# Домашнее задание к занятию «Репликация и масштабирование. Часть 1» - `Корниенко Кирилл`


### Задание 1. 
На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

### *Ответ:*

Главное различие между Master-Slave и Master-Master заключается в том, кому разрешено изменять данные и как система обрабатывает конфликты.
### Режим Master-Slave (Ведущий-Ведомый)
В этой архитектуре один сервер назначается главным (Master), а остальные — подчиненными (Slaves).

### Запись (Write): 
Только Master может принимать запросы на запись (INSERT, UPDATE, DELETE). Все изменения данных происходят исключительно на нем.

### Чтение (Read): 
Чтение данных можно распределять между Master и/или Slaves. Обычно всю нагрузку по чтению отдают Slaves, чтобы разгрузить Master для операций записи.

### Поток данных: 
Master записывает все изменения в специальный бинарный лог (журнал). Каждый Slave подключается к Masterу, читает этот лог и применяет те же изменения к себе, оставаясь точной копией.

Из плюсов можно выделить отсутствие конфликтов, легкую масштабируемость чтения (добавив еще один Slave сервер), создание резервной копии без остановки Master сервера

Из минусов - единая точка отказа, если Master сервер выходит из строя то запись данных в базу становится невозможна.

### Режим Master-Master (Ведущий-Ведущий / Мульти-мастер)
В этой архитектуре два или более серверов являются равноправными.

### Запись (Write): 
Любой из Master-серверов может принимать запросы на запись. Клиент может писать на любой узел.

### Чтение (Read): 
Как и в первом случае, чтение можно делать с любого узла.

### Поток данных: 
Каждый Master реплицирует свои изменения на другой Master (или все остальные). Все серверы одновременно выступают и в роли Masterа (для клиентов) и в роли Slave (друг для друга).

Из плюсов можно выделить отказоустойчивость

Из минусов - возможные конфликты данных


### Задание 2. 

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

### Ответ

1. Подготовка рабочей директории

```bash
# Создаем папку для проекта
mkdir ~/mysql-replication
cd ~/mysql-replication
# Создаем пустые файлы
touch Dockerfile_master Dockerfile_slave master.cnf master.sql slave.cnf slave.sql
```

2. Редактирование конфигурационных файлов
   
   ![replic1](https://github.com/kirill-kornienko/Replic1/blob/main/replic1.png)
   
   ![replic2](https://github.com/kirill-kornienko/Replic1/blob/main/replic2.png)
   
   ![replic3](https://github.com/kirill-kornienko/Replic1/blob/main/replic3.png)
   
   ![replic4](https://github.com/kirill-kornienko/Replic1/blob/main/replic4.png)
   
   ![replic5](https://github.com/kirill-kornienko/Replic1/blob/main/replic5.png)
   
   ![replic6](https://github.com/kirill-kornienko/Replic1/blob/main/replic6.png)
   

4. Сборка Docker- образов
   
```bash
# Собираем образ мастера
docker build -t mysql_master -f Dockerfile_master .

# Собираем образ слейва
docker build -t mysql_slave -f Dockerfile_slave .
```

4. Создаем сети Docker
   
```bash
# Создаем сеть для репликации
docker network create replication

# Проверяем, что сеть создалась
docker network ls
```

![docker_networks](https://github.com/kirill-kornienko/Replic1/blob/main/docker%20network.png)

5. Запускаем контейнеры на свободных портах (порт 3306 занят локаным MySQL)
   
```bash
# Запускаем мастер на порту 33061 
sudo docker run -d \
  --name mysql_master \
  --net replication \
  -p 33061:3306 \
  mysql_master

# Запускаем слейв на порту 33062 
sudo docker run -d \
  --name mysql_slave \
  --net replication \
  -p 33062:3306 \
  mysql_slave
# Проверяем, что оба контейнера запущены
docker ps
```

![docker_ps](https://github.com/kirill-kornienko/Replic1/blob/main/docker%20ps.png)

6. Проверка репликации
   
Проверяем статус репликации на слейве

```bash
# Подключаемся к слейву
docker exec -it mysql_slave mysql -u root -p
```

```sql
SHOW REPLICA STATUS\G
```

![replica_status](https://github.com/kirill-kornienko/Replic1/blob/main/replica_status.png)

7. Тестирование репликации

   Создаем новые данные на мастере

```bash
# Подключаемся к мастеру
sudo docker exec -it mysql_master mysql -u root -p
```

```sql
Используем существующую базу
USE test_db;

-- Добавляем новые записи (с другими ID)
INSERT INTO test_table VALUES (2, 'Second Record');
INSERT INTO test_table VALUES (3, 'Third Record');

-- Проверяем все записи
SELECT * FROM test_table;
```

![create_database_master](https://github.com/kirill-kornienko/Replic1/blob/main/create_database_master.png)

Проверяем данные на слейве

```bash
# Подключаемся к слейву
docker exec -it mysql_slave mysql -u root -p
```

```sql
SHOW DATABASES;
USE test_db;
SHOW TABLES;
SELECT * FROM test_table;
```

![show_database_slave](https://github.com/kirill-kornienko/Replic1/blob/main/show_database_slave.png)

8. Останавливаем контейнеры

```bash
docker stop mysql_master mysql_slave
```

1{docker_stop](https://github.com/kirill-kornienko/Replic1/blob/main/docker_stop.png)









