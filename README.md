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


*Приведите скриншот команды 'curl -X GET 'localhost:9200/_cluster/health?pretty', сделанной на сервере с установленным Elasticsearch. Где будет виден нестандартный cluster_name.*.

### *Ответ:*

![elasticsearch](https://github.com/kirill-kornienko/ELK/blob/main/elasticsearch.png)


### Задание 2. Kibana

Установите и запустите Kibana.

*Приведите скриншот интерфейса Kibana на странице http://<ip вашего сервера>:5601/app/dev_tools#/console, где будет выполнен запрос GET /_cluster/health?pretty.*.


### *Ответ:*
![kibana](https://github.com/kirill-kornienko/ELK/blob/main/kibana.png)


### Задание 3. Logstash

Установите и запустите Logstash и Nginx. С помощью Logstash отправьте access-лог Nginx в Elasticsearch.


*Приведите скриншот интерфейса Kibana, на котором видны логи Nginx.*.


### *Ответ*:
![logstash](https://github.com/kirill-kornienko/ELK/blob/main/logstash.png)


### Задание 4. Filebeat.

Установите и запустите Filebeat. Переключите поставку логов Nginx с Logstash на Filebeat.

*Приведите скриншот интерфейса Kibana, на котором видны логи Nginx, которые были отправлены через Filebeat.*.

### *Ответ*:

![filebeat](https://github.com/kirill-kornienko/ELK/blob/main/filebeat.png)



