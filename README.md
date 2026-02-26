# Домашнее задание к занятию «Репликация и масштабирование. Часть 2» - `Корниенко Кирилл`


### Задание 1. 
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

### *Ответ:*

Master-Slave масштабирование: основные плюсы

1. Master + один Slave (Passive)

Отказоустойчивость: Есть "горячая" копия на случай падения основного сервера.

Резервное копирование: Можно делать бэкапы с реплики, не нагружая основной сервер.

Обслуживание: Возможность обновлять или чинить slave без простоя системы.

2. Master + много Slaves

Масштабирование чтения: Основная нагрузка (чтение данных) распределяется между несколькими серверами, что повышает производительность.

География: Можно разнести серверы по разным странам для ускорения доступа локальных пользователей.

Специализация: Разные реплики можно использовать под разные задачи: одну под аналитику,

### Главный минус обеих схем — 
Задержка репликации . Если данные критически важны и должны быть актуальны «прямо сейчас», читать их нужно обязательно с master-сервера. Slave-серверы могут показывать устаревшие данные на доли секунды или даже секунды, если нагрузка велика

### Задание 2. 

Разработайте план для выполнения горизонтального и вертикального шаринга базы данных. База данных состоит из трёх таблиц:

- пользователи,
- книги,
- магазины (столбцы произвольно).
  
Опишите принципы построения системы и их разграничение или разбивку между базами данных.

*Пришлите блоксхему, где и что будет располагаться. Опишите, в каких режимах будут работать сервера.*

### Ответ

Архитектура решения

Cоздадим:

Вертикальный шардинг: разные таблицы на разных серверах

Горизонтальный шардинг: таблица users разбита по user_id (3 шарда)

Мастер-сервер: единая точка входа с представлением (VIEW) и правилами маршрутизации

1. Создание структуры проекта

```bash
# Создаем директорию проекта
mkdir ~/sharding-project
cd ~/sharding-project

# Создаем структуру папок
mkdir -p conf/{master,shard1,shard2,shard3}
mkdir -p data/{master,shard1,shard2,shard3}
```
2. Создаем Docker Compose файл

```bash
nano docker-compose.yml:
```

```yaml
version: '3.8'

networks:
  sharding_network:
    driver: bridge

services:
  # Мастер-сервер (точка входа для приложения)
  master:
    image: postgres:15
    container_name: postgres_master
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"
    volumes:
      - ./conf/master/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./data/master:/var/lib/postgresql/data
    networks:
      - sharding_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Шард 1 (пользователи с user_id % 3 = 0)
  shard1:
    image: postgres:15
    container_name: postgres_shard1
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: books
    ports:
      - "5433:5432"
    volumes:
      - ./conf/shard1/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./data/shard1:/var/lib/postgresql/data
    networks:
      - sharding_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Шард 2 (пользователи с user_id % 3 = 1)
  shard2:
    image: postgres:15
    container_name: postgres_shard2
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: books
    ports:
      - "5434:5432"
    volumes:
      - ./conf/shard2/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./data/shard2:/var/lib/postgresql/data
    networks:
      - sharding_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Шард 3 (пользователи с user_id % 3 = 2)
  shard3:
    image: postgres:15
    container_name: postgres_shard3
    restart: unless-stopped
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: books
    ports:
      - "5435:5432"
    volumes:
      - ./conf/shard3/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./data/shard3:/var/lib/postgresql/data
    networks:
      - sharding_network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

3. Создаем SQL скрипты для инициализации
   
Скрипты для шардов

```bash
   nano conf/shard1/init.sql
```

```sql
  -- Создаем таблицу пользователей на шарде
CREATE TABLE IF NOT EXISTS users (
    user_id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    country VARCHAR(50),
    registration_date DATE DEFAULT CURRENT_DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Создаем таблицу книг (вертикальный шард)
CREATE TABLE IF NOT EXISTS books (
    book_id BIGSERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(100) NOT NULL,
    isbn VARCHAR(20),
    price DECIMAL(10, 2),
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Создаем таблицу магазинов (вертикальный шард)
CREATE TABLE IF NOT EXISTS shops (
    shop_id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address TEXT,
    phone VARCHAR(20),
    country VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Добавляем ограничение CHECK для горизонтального шардинга users
-- Шард 1: user_id % 3 = 0
ALTER TABLE users ADD CONSTRAINT users_shard_check 
    CHECK (user_id % 3 = 0);

-- Индексы для ускорения
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_country ON users(country);
CREATE INDEX idx_books_category ON books(category_id);
CREATE INDEX idx_shops_country ON shops(country);

-- Вставляем тестовые данные для демонстрации
-- Пользователи (только те, что подходят для этого шарда)
INSERT INTO users (user_id, name, email, country) VALUES
    (3, 'Пользователь 3', 'user3@mail.com', 'Россия'),
    (6, 'Пользователь 6', 'user6@mail.com', 'США'),
    (9, 'Пользователь 9', 'user9@mail.com', 'Германия')
ON CONFLICT (user_id) DO NOTHING;

-- Книги (одинаковые на всех шардах - вертикальный шардинг)
INSERT INTO books (title, author, price, category_id) VALUES
    ('Война и мир', 'Лев Толстой', 500.00, 1),
    ('Преступление и наказание', 'Федор Достоевский', 450.00, 1),
    ('1984', 'Джордж Оруэлл', 400.00, 2)
ON CONFLICT (book_id) DO NOTHING;

-- Магазины (одинаковые на всех шардах - вертикальный шардинг)
INSERT INTO shops (name, address, country) VALUES
    ('Читай-город', 'ул. Ленина, 10', 'Россия'),
    ('Книжный Лабиринт', 'пр. Мира, 25', 'Россия'),
    ('Book Depository', 'Baker Street, 221', 'Великобритания')
ON CONFLICT (shop_id) DO NOTHING;

```

```bash
nano conf/shard2/init.sql
```

```sql
-- Создаем таблицу пользователей на шарде
CREATE TABLE IF NOT EXISTS users (
    user_id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    country VARCHAR(50),
    registration_date DATE DEFAULT CURRENT_DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Создаем таблицу книг (вертикальный шард)
CREATE TABLE IF NOT EXISTS books (
    book_id BIGSERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(100) NOT NULL,
    isbn VARCHAR(20),
    price DECIMAL(10, 2),
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Создаем таблицу магазинов (вертикальный шард)
CREATE TABLE IF NOT EXISTS shops (
    shop_id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address TEXT,
    phone VARCHAR(20),
    country VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Добавляем ограничение CHECK для горизонтального шардинга users
-- Шард 2: user_id % 3 = 1
ALTER TABLE users ADD CONSTRAINT users_shard_check 
    CHECK (user_id % 3 = 1);

-- Индексы для ускорения
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_country ON users(country);
CREATE INDEX idx_books_category ON books(category_id);
CREATE INDEX idx_shops_country ON shops(country);

-- Вставляем тестовые данные для демонстрации
-- Пользователи (только те, что подходят для этого шарда)
INSERT INTO users (user_id, name, email, country) VALUES
    (1, 'Пользователь 1', 'user1@mail.com', 'Россия'),
    (4, 'Пользователь 4', 'user4@mail.com', 'США'),
    (7, 'Пользователь 7', 'user7@mail.com', 'Германия')
ON CONFLICT (user_id) DO NOTHING;

-- Книги (одинаковые на всех шардах - вертикальный шардинг)
INSERT INTO books (title, author, price, category_id) VALUES
    ('Война и мир', 'Лев Толстой', 500.00, 1),
    ('Преступление и наказание', 'Федор Достоевский', 450.00, 1),
    ('1984', 'Джордж Оруэлл', 400.00, 2)
ON CONFLICT (book_id) DO NOTHING;

-- Магазины (одинаковые на всех шардах - вертикальный шардинг)
INSERT INTO shops (name, address, country) VALUES
    ('Читай-город', 'ул. Ленина, 10', 'Россия'),
    ('Книжный Лабиринт', 'пр. Мира, 25', 'Россия'),
    ('Book Depository', 'Baker Street, 221', 'Великобритания')
ON CONFLICT (shop_id) DO NOTHING;
```

```bash
nano conf/shard3/init.sql
```

```sql
-- Создаем таблицу пользователей на шарде
CREATE TABLE IF NOT EXISTS users (
    user_id BIGINT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    country VARCHAR(50),
    registration_date DATE DEFAULT CURRENT_DATE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Создаем таблицу книг (вертикальный шард)
CREATE TABLE IF NOT EXISTS books (
    book_id BIGSERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    author VARCHAR(100) NOT NULL,
    isbn VARCHAR(20),
    price DECIMAL(10, 2),
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Создаем таблицу магазинов (вертикальный шард)
CREATE TABLE IF NOT EXISTS shops (
    shop_id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    address TEXT,
    phone VARCHAR(20),
    country VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Добавляем ограничение CHECK для горизонтального шардинга users
-- Шард 3: user_id % 3 = 2
ALTER TABLE users ADD CONSTRAINT users_shard_check 
    CHECK (user_id % 3 = 2);

-- Индексы для ускорения
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_country ON users(country);
CREATE INDEX idx_books_category ON books(category_id);
CREATE INDEX idx_shops_country ON shops(country);

-- Вставляем тестовые данные для демонстрации
-- Пользователи (только те, что подходят для этого шарда)
INSERT INTO users (user_id, name, email, country) VALUES
    (2, 'Пользователь 2', 'user2@mail.com', 'Россия'),
    (5, 'Пользователь 5', 'user5@mail.com', 'США'),
    (8, 'Пользователь 8', 'user8@mail.com', 'Германия')
ON CONFLICT (user_id) DO NOTHING;

-- Книги (одинаковые на всех шардах - вертикальный шардинг)
INSERT INTO books (title, author, price, category_id) VALUES
    ('Война и мир', 'Лев Толстой', 500.00, 1),
    ('Преступление и наказание', 'Федор Достоевский', 450.00, 1),
    ('1984', 'Джордж Оруэлл', 400.00, 2)
ON CONFLICT (book_id) DO NOTHING;

-- Магазины (одинаковые на всех шардах - вертикальный шардинг)
INSERT INTO shops (name, address, country) VALUES
    ('Читай-город', 'ул. Ленина, 10', 'Россия'),
    ('Книжный Лабиринт', 'пр. Мира, 25', 'Россия'),
    ('Book Depository', 'Baker Street, 221', 'Великобритания')
ON CONFLICT (shop_id) DO NOTHING;
```

Создаем скрипт для мастер сервера

```bash
nano conf/master/init.sql
```

```sql
-- Включаем расширение для внешних данных
CREATE EXTENSION IF NOT EXISTS postgres_fdw;

-- ============================================
-- 1. ПОДКЛЮЧЕНИЕ К ШАРДАМ (HORIZONTAL SHARDING для users)
-- ============================================

-- Сервер для шарда 1 (user_id % 3 = 0)
CREATE SERVER IF NOT EXISTS shard1_server
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'shard1', port '5432', dbname 'books');

CREATE USER MAPPING IF NOT EXISTS FOR postgres
    SERVER shard1_server
    OPTIONS (user 'postgres', password 'postgres');

-- Сервер для шарда 2 (user_id % 3 = 1)
CREATE SERVER IF NOT EXISTS shard2_server
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'shard2', port '5432', dbname 'books');

CREATE USER MAPPING IF NOT EXISTS FOR postgres
    SERVER shard2_server
    OPTIONS (user 'postgres', password 'postgres');

-- Сервер для шарда 3 (user_id % 3 = 2)
CREATE SERVER IF NOT EXISTS shard3_server
    FOREIGN DATA WRAPPER postgres_fdw
    OPTIONS (host 'shard3', port '5432', dbname 'books');

CREATE USER MAPPING IF NOT EXISTS FOR postgres
    SERVER shard3_server
    OPTIONS (user 'postgres', password 'postgres');

-- ============================================
-- 2. СОЗДАНИЕ ВНЕШНИХ ТАБЛИЦ (FOREIGN TABLES)
-- ============================================

-- Внешняя таблица для users с шарда 1
CREATE FOREIGN TABLE IF NOT EXISTS users_shard1 (
    user_id BIGINT,
    name VARCHAR(100),
    email VARCHAR(100),
    country VARCHAR(50),
    registration_date DATE,
    created_at TIMESTAMP
) SERVER shard1_server
OPTIONS (schema_name 'public', table_name 'users');

-- Внешняя таблица для users с шарда 2
CREATE FOREIGN TABLE IF NOT EXISTS users_shard2 (
    user_id BIGINT,
    name VARCHAR(100),
    email VARCHAR(100),
    country VARCHAR(50),
    registration_date DATE,
    created_at TIMESTAMP
) SERVER shard2_server
OPTIONS (schema_name 'public', table_name 'users');

-- Внешняя таблица для users с шарда 3
CREATE FOREIGN TABLE IF NOT EXISTS users_shard3 (
    user_id BIGINT,
    name VARCHAR(100),
    email VARCHAR(100),
    country VARCHAR(50),
    registration_date DATE,
    created_at TIMESTAMP
) SERVER shard3_server
OPTIONS (schema_name 'public', table_name 'users');

-- ============================================
-- 3. ВЕРТИКАЛЬНЫЙ ШАРДИНГ (разные таблицы на разных серверах)
-- ============================================

-- Для простоты используем шард 1 для книг
CREATE FOREIGN TABLE IF NOT EXISTS books (
    book_id BIGINT,
    title VARCHAR(200),
    author VARCHAR(100),
    isbn VARCHAR(20),
    price DECIMAL(10,2),
    category_id INT,
    created_at TIMESTAMP
) SERVER shard1_server
OPTIONS (schema_name 'public', table_name 'books');

-- Используем шард 2 для магазинов
CREATE FOREIGN TABLE IF NOT EXISTS shops (
    shop_id BIGINT,
    name VARCHAR(100),
    address TEXT,
    phone VARCHAR(20),
    country VARCHAR(50),
    created_at TIMESTAMP
) SERVER shard2_server
OPTIONS (schema_name 'public', table_name 'shops');

-- ============================================
-- 4. СОЗДАНИЕ ПРЕДСТАВЛЕНИЙ (VIEWS) ДЛЯ ЕДИНОЙ ТОЧКИ ДОСТУПА
-- ============================================

-- Общее представление для пользователей (объединяет все шарды)
CREATE VIEW users AS
    SELECT * FROM users_shard1
    UNION ALL
    SELECT * FROM users_shard2
    UNION ALL
    SELECT * FROM users_shard3;

-- ============================================
-- 5. ПРАВИЛА ДЛЯ АВТОМАТИЧЕСКОЙ МАРШРУТИЗАЦИИ INSERT
-- ============================================

-- Функция для определения шарда по user_id
CREATE OR REPLACE FUNCTION get_shard_for_user(user_id BIGINT)
RETURNS TEXT AS $$
BEGIN
    CASE user_id % 3
        WHEN 0 THEN RETURN 'shard1';
        WHEN 1 THEN RETURN 'shard2';
        WHEN 2 THEN RETURN 'shard3';
    END CASE;
END;
$$ LANGUAGE plpgsql;

-- Правило для INSERT в users (горизонтальный шардинг)
CREATE OR REPLACE RULE users_insert AS ON INSERT TO users
DO INSTEAD (
    CASE (NEW.user_id % 3)
        WHEN 0 THEN INSERT INTO users_shard1 VALUES (NEW.*);
        WHEN 1 THEN INSERT INTO users_shard2 VALUES (NEW.*);
        WHEN 2 THEN INSERT INTO users_shard3 VALUES (NEW.*);
    END CASE
);

-- Правило для UPDATE в users
CREATE OR REPLACE RULE users_update AS ON UPDATE TO users
DO INSTEAD (
    CASE (OLD.user_id % 3)
        WHEN 0 THEN UPDATE users_shard1 SET 
            name = NEW.name, 
            email = NEW.email,
            country = NEW.country,
            registration_date = NEW.registration_date
            WHERE user_id = OLD.user_id;
        WHEN 1 THEN UPDATE users_shard2 SET 
            name = NEW.name, 
            email = NEW.email,
            country = NEW.country,
            registration_date = NEW.registration_date
            WHERE user_id = OLD.user_id;
        WHEN 2 THEN UPDATE users_shard3 SET 
            name = NEW.name, 
            email = NEW.email,
            country = NEW.country,
            registration_date = NEW.registration_date
            WHERE user_id = OLD.user_id;
    END CASE
);

-- Правило для DELETE в users
CREATE OR REPLACE RULE users_delete AS ON DELETE TO users
DO INSTEAD (
    CASE (OLD.user_id % 3)
        WHEN 0 THEN DELETE FROM users_shard1 WHERE user_id = OLD.user_id;
        WHEN 1 THEN DELETE FROM users_shard2 WHERE user_id = OLD.user_id;
        WHEN 2 THEN DELETE FROM users_shard3 WHERE user_id = OLD.user_id;
    END CASE
);

-- ============================================
-- 6. ТЕСТОВЫЕ ДАННЫЕ
-- ============================================

-- Вставка пользователей через мастер (автоматически распределятся по шардам)
INSERT INTO users (user_id, name, email, country) VALUES
    (1, 'Иван Петров', 'ivan@mail.com', 'Россия'),
    (2, 'Мария Смирнова', 'maria@mail.com', 'Россия'),
    (3, 'John Smith', 'john@email.com', 'США'),
    (4, 'Anna Schmidt', 'anna@email.com', 'Германия'),
    (5, 'Pierre Dubois', 'pierre@email.com', 'Франция'),
    (6, 'Carlos Garcia', 'carlos@email.com', 'Испания')
ON CONFLICT DO NOTHING;

-- Проверка распределения
SELECT 'Users in shard1 (user_id % 3 = 0):' as info;
SELECT * FROM users_shard1;

SELECT 'Users in shard2 (user_id % 3 = 1):' as info;
SELECT * FROM users_shard2;

SELECT 'Users in shard3 (user_id % 3 = 2):' as info;
SELECT * FROM users_shard3;
```

4. Запуск систем

```bash
docker compose up -d

# Проверка статуса
docker compose ps
```

![docker_ps](https://github.com/kirill-kornienko/Replic2/blob/main/docker_up.png)

Подключение к мастер серверу

```bash
# Подключаемся к мастеру
docker exec -it postgres_master psql -U postgres
```

Проверка распределения данных

```sql
-- Смотрим всех пользователей через единое представление
SELECT * FROM users ORDER BY user_id;

-- Проверяем, на каких шардах лежат пользователи
SELECT 'Шард 1 (user_id % 3 = 0):' as info;
SELECT * FROM users_shard1;

SELECT 'Шард 2 (user_id % 3 = 1):' as info;
SELECT * FROM users_shard2;

SELECT 'Шард 3 (user_id % 3 = 2):' as info;
SELECT * FROM users_shard3;
```

![users](https://github.com/kirill-kornienko/Replic2/blob/main/users.png)

Тестирование вставки новых данных

```sql
-- Вставляем нового пользователя
INSERT INTO users (user_id, name, email, country) 
VALUES (7, 'Новый пользователь', 'new@mail.com', 'Россия');

-- Проверяем, куда он попал
SELECT user_id, user_id % 3 as shard_number, * 
FROM users_shard2 
WHERE user_id = 7;  -- 7 % 3 = 1, должен быть в shard2
```

![test_insert](https://github.com/kirill-kornienko/Replic2/blob/main/test_insert.png)

Проверка вертикального шардинга:

```sql
-- Книги должны быть на shard1
SELECT * FROM books;

-- Магазины должны быть на shard2
SELECT * FROM shops;
```

![test_shard](https://github.com/kirill-kornienko/Replic2/blob/main/test_books_shops.png)

5. Блок-схема архитектуры

```text
┌─────────────────────────────────────────────────────────────────────┐
│                        КЛИЕНТ / ПРИЛОЖЕНИЕ                           │
│              (подключается к мастеру на порт 5432)                   │
└───────────────────────────────────┬─────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         МАСТЕР-СЕРВЕР (postgres_master)              │
│                              (порт 5432)                              │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                      ПРЕДСТАВЛЕНИЕ (VIEW)                     │    │
│  │                         users                                  │    │
│  │  (объединяет данные со всех шардов через UNION ALL)           │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              ПРАВИЛА МАРШРУТИЗАЦИИ (RULES)                   │    │
│  │  INSERT → определяет шард по user_id % 3                     │    │
│  │  UPDATE → обновляет на соответствующем шарде                 │    │
│  │  DELETE → удаляет с соответствующего шарда                   │    │
│  └─────────────────────────────────────────────────────────────┘    │
│                                                                       │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │              ВНЕШНИЕ ТАБЛИЦЫ (FOREIGN TABLES)               │    │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │    │
│  │  │users_    │  │users_    │  │users_    │  │ books    │   │    │
│  │  │shard1    │  │shard2    │  │shard3    │  │(shard1)  │   │    │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘   │    │
│  │       │             │             │             │          │    │
│  │       │             │             │             │  ┌──────────┐ │
│  │       │             │             │             │  │ shops    │ │
│  │       │             │             │             │  │(shard2)  │ │
│  │       │             │             │             │  └────┬─────┘ │
│  └───────┼─────────────┼─────────────┼─────────────┼──────┼───────┘
└──────────┼─────────────┼─────────────┼─────────────┼──────┼─────────
           │             │             │             │      │
           ▼             ▼             ▼             ▼      ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  ШАРД 1         │ │  ШАРД 2         │ │  ШАРД 3         │ │  ВЕРТИКАЛЬНЫЕ   │
│  (порт 5433)    │ │  (порт 5434)    │ │  (порт 5435)    │ │    ШАРДЫ        │
│                 │ │                 │ │                 │ │                 │
│  Таблица users  │ │  Таблица users  │ │  Таблица users  │ │  books (shard1) │
│  CHECK:         │ │  CHECK:         │ │  CHECK:         │ │  shops (shard2) │
│  user_id % 3=0  │ │  user_id % 3=1  │ │  user_id % 3=2  │ │                 │
│                 │ │                 │ │                 │ │                 │
│  Также:         │ │  Также:         │ │  Также:         │ │                 │
│  - books        │ │  - shops        │ │  - (нет         │ │                 │
│                 │ │                 │ │    вертикальных)│ │                 │
└─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────────┘
```

6. Режим работы серверов

```text
Сервер	Тип шардинга	                                  Что хранит	                                             Режим работы
master	Центральный маршрутизатор       	              Нет данных, только внешние таблицы и правила	           Proxy / Router (принимает все запросы, перенаправляет на шарды)
shard1	Горизонтальный (users) + Вертикальный (books)	  users (user_id % 3 = 0), books	                         Data node (чтение/запись своих данных)
shard2	Горизонтальный (users) + Вертикальный (shops)	  users (user_id % 3 = 1), shops	                         Data node (чтение/запись своих данных)
shard3	Горизонтальный (users)	                          users (user_id % 3 = 2)	                                 Data node (чтение/запись своих данных)
```

   
