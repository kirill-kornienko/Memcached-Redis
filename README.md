# Домашнее задание к занятию «Работа с данными (DDL/DML)» - `Корниенко Кирилл`


### Задание 1. 
1.1. Поднимите чистый инстанс MySQL версии 8.0+. Можно использовать локальный сервер или контейнер Docker.

1.2. Создайте учётную запись sys_temp.

1.3. Выполните запрос на получение списка пользователей в базе данных. (скриншот)

1.4. Дайте все права для пользователя sys_temp.

1.5. Выполните запрос на получение списка прав для пользователя sys_temp. (скриншот)

1.6. Переподключитесь к базе данных от имени sys_temp.

Для смены типа аутентификации с sha2 используйте запрос:

```
ALTER USER 'sys_test'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
1.6. По ссылке https://downloads.mysql.com/docs/sakila-db.zip скачайте дамп базы данных.

1.7. Восстановите дамп в базу данных.

1.8. При работе в IDE сформируйте ER-диаграмму получившейся базы данных. При работе в командной строке используйте команду для получения всех таблиц базы данных. (скриншот)

*Результатом работы должны быть скриншоты обозначенных заданий, а также простыня со всеми запросами.

### *Ответ:*

1.1. Поднимите чистый инстанс MySQL версии 8.0+. Можно использовать локальный сервер или контейнер Docker.

```
wget -c https://dev.mysql.com/get/mysql-apt-config_0.8.28-1_all.deb
sudo dpkg -i mysql-apt-config_0.8.28-1_all.deb
sudo apt update
sudo apt install mysql-server
sudo systemctl status mysql
sudo systemctl enable mysql
mysql -u root -p
```
![status_mysql](https://github.com/kirill-kornienko/DDL-DML/blob/main/status%20mysql.png)
![mysql](https://github.com/kirill-kornienko/DDL-DML/blob/main/mysql.png)

1.2. Создайте учётную запись sys_temp.
```
CREATE USER 'sys_temp'@'localhost' IDENTIFIED BY '1234567';
```
![create_user](https://github.com/kirill-kornienko/DDL-DML/blob/main/create%20user.png)

1.3. Выполните запрос на получение списка пользователей в базе данных. (скриншот)

```
SELECT user FROM mysql.user;

```

![select_user](https://github.com/kirill-kornienko/DDL-DML/blob/main/select%20user.png)

1.4. Дайте все права для пользователя sys_temp.

```
GRANT ALL PRIVILEGES ON *.* TO 'sys_temp'@'localhost' WITH GRANT OPTION;
```
![all_priveleges](https://github.com/kirill-kornienko/DDL-DML/blob/main/all_priveleges.png)

1.5. Выполните запрос на получение списка прав для пользователя sys_temp. (скриншот)

```
SELECT * FROM information_schema.user_privileges WHERE GRANTEE="'sys_temp'@'localhost'";
```
![systemp1](https://github.com/kirill-kornienko/DDL-DML/blob/main/sys_temp1.png)
![systemp2](https://github.com/kirill-kornienko/DDL-DML/blob/main/sys_temp2.png)

1.6. Переподключитесь к базе данных от имени sys_temp.

```bash
SYSTEM mysql -u sys_temp -p
SELECT user();
```
![select_user](https://github.com/kirill-kornienko/DDL-DML/blob/main/user%20sys_temp.png)

Для смены типа аутентификации с sha2 используйте запрос:

```
ALTER USER 'sys_test'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```
1.7. По ссылке [https://downloads.mysql.com/docs/sakila-db.zip](https://downloads.mysql.com/docs/sakila-db.zip) скачайте дамп базы данных.

```
wget https://downloads.mysql.com/docs/sakila-db.zip
unzip sakila-db.zip
```
![sakila](https://github.com/kirill-kornienko/DDL-DML/blob/main/sakila.png)

1.8. Восстановите дамп в базу данных.

```
source /home/tverdyakov/sakila-db/sakila-schema.sql
source /home/tverdyakov/sakila-db/sakila-data.sql
SHOW DATABASES;
```
![sakila_schema](https://github.com/kirill-kornienko/DDL-DML/blob/main/sakila_schema1.png)
![sakila_data](https://github.com/kirill-kornienko/DDL-DML/blob/main/sakila_data.png)
![show_databases](https://github.com/kirill-kornienko/DDL-DML/blob/main/show_database.png)

1.9. При работе в IDE сформируйте ER-диаграмму получившейся базы данных. При работе в командной строке используйте команду для получения всех таблиц базы данных. (скриншот)

```
SHOW TABLES;
```
![show_tables](https://github.com/kirill-kornienko/DDL-DML/blob/main/show_tables.png)

### Задание 2. 

Составьте таблицу, используя любой текстовый редактор или Excel, в которой должно быть два столбца: в первом должны быть названия таблиц восстановленной базы, во втором названия первичных ключей этих таблиц. Пример: (скриншот/текст)

```
Название таблицы | Название первичного ключа
customer         | customer_id
```
### *Ответ:*
```
+---------------+--------------+
| TABLE_NAME    | COLUMN_NAME  |
+---------------+--------------+
| actor         | actor_id     |
| address       | address_id   |
| category      | category_id  |
| city          | city_id      |
| country       | country_id   |
| customer      | customer_id  |
| film          | film_id      |
| film_actor    | actor_id     |
| film_actor    | film_id      |
| film_category | film_id      |
| film_category | category_id  |
| film_text     | film_id      |
| inventory     | inventory_id |
| language      | language_id  |
| payment       | payment_id   |
| rental        | rental_id    |
| staff         | staff_id     |
| store         | store_id     |
+---------------+--------------+
```

![table](https://github.com/kirill-kornienko/DDL-DML/blob/main/2.tables.png)



