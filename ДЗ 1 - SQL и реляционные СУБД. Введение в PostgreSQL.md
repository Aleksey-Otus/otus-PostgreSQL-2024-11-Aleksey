## создать новый проект в Яндекс облако или на любых ВМ, докере
  * https://center.yandex.cloud/  
## далее создать инстанс виртуальной машины с дефолтными параметрами
  Создать сеть:
  Каталог: default
  Имя: otus-net-88
  Подсеть: otus-net-88-ru-central1-b
## добавить свой ssh ключ в metadata ВМ
ssh-keygen -t rsa -b 2048
name ssh-key: yc_key
пароль sh-key: 123
notepad yc_key.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC33ottJ45e8m8XegwuIuVATrn9X+4t+2t93BK6GhSv9pVHZJIFYn0uR5gT9cHAPnPEkj573SiHFcrfhvYKoiivX/OYvRraXVcL8fN3J8+BhKAme5jnRM95DPvZlWxJ3QwPAff6sI51Rbz+W6Em1MZ4nDhaQuVlm4SZQ0t/hK11FbqIgopYTkELPjmx31y8LgECVrMD2oZ1xNenBq0XzLdiR+aLH8ao/to68Cgv7pfZvILd06t7wtSQ2EA07bnQAPmwMuhDDYN8aIUVwCa/0d0KZfTaiApBLPCxzv/uN3iKcVxv1pePyJofTUMICmeZ9MgrQIIcV+C74BTZfrx7K94L support@DESKTOP-C1VUI47
Логин: otus
Имя SSH-ключа: ssh-key
Имя ВМ: compute-vm-2-2-20-hdd-1739048396829

Сетевой интерфейс
Внутренний IPv4-адрес: 10.129.0.6
Публичный IPv4-адрес: 89.169.166.154
Тип публичного IPv4-адреса: Динамический
Подсеть: otus-net-88-ru-central1-b
Группа безопасности: default-sg-enpls4kbfof0ttshsj0c

## зайти удаленным ssh (первая сессия), не забывайте про ssh-add
Виртуальные машины: https://console.yandex.cloud/folders/b1g217bjlm0a86q0meib/compute/instances
Подключение к виртуальной машине 
ssh -i ~/yc_key otus@89.169.166.154 ## меняется при каждом новом подключении

## поставить PostgreSQL
Установка Postgres:
sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql

pg_ctlcluster
Ver Cluster Port Status Owner    Data directory              Log file
17  main    5432 online postgres /var/lib/postgresql/17/main /var/log/postgresql/postgresql-17-main.log

## устанавливаем пароль psql
\password 123
\q

## Добавить сетевые правила для подключения:
sudo nano /etc/postgresql/17/main/postgresql.com
listen_addresses = '*'

sudo nano /etc/postgresql/17/main/pg_hba.conf
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             0.0.0.0/0            scram-sha-256
sudo pg_ctlcluster 17 main restart
pg_ctlcluster

## зайти вторым ssh (вторая сессия)
запускаю второй Windows PowerShell
ssh -i ~/yc_key otus@89.169.166.154

## запустить везде psql из под пользователя postgres
sudo -u postgres psql

## выключить auto commit
\set AUTOCOMMIT off
\set autocommit = off;

сделать

## в первой сессии новую таблицу и наполнить ее данными 
create table persons(id serial, first_name text, second_name text); 
insert into persons(first_name, second_name) values('ivan', 'ivanov'); 
insert into persons(first_name, second_name) values('petr', 'petrov'); 
commit;

postgres=# select id serial, first_name text, second_name text from persons;
 serial |  text  |  text
--------+--------+---------
      1 | ivan   | ivanov
      2 | petr   | petrov
(2 rows)

## посмотреть текущий уровень изоляции: show transaction isolation level
SHOW TRANSACTION ISOLATION LEVEL;
postgres=*# SHOW TRANSACTION ISOLATION LEVEL;
 transaction_isolation
-----------------------
 read committed
(1 row)
## начать новую транзакцию в обоих сессиях с дефолтным (не меняя) уровнем изоляции
## в первой сессии добавить новую запись 
insert into persons(first_name, second_name) values('sergey', 'sergeev');

postgres=# select id serial, first_name text, second_name text from persons;
 serial |  text  |  text
--------+--------+---------
      1 | ivan   | ivanov
      2 | petr   | petrov
      3 | sergey | sergeev
(3 rows)

## сделать select from persons во второй сессии
select id serial, first_name text, second_name text from persons;

postgres=# select id serial, first_name text, second_name text from persons;
 serial |  text  |  text
--------+--------+---------
      1 | ivan   | ivanov
      2 | petr   | petrov
(2 rows)

## видите ли вы новую запись и если да то почему?
новой записи нет тк нет commit
## завершить первую транзакцию - commit;
## сделать select from persons во второй сессии
postgres=# select id serial, first_name text, second_name text from persons;
 serial |  text  |  text
--------+--------+---------
      1 | ivan   | ivanov
      2 | petr   | petrov
      3 | sergey | sergeev
(3 rows)
## видите ли вы новую запись и если да то почему?
новая запись видна тк применили commit для успешного завершения транзакции
## завершите транзакцию во второй сессии
## начать новые но уже repeatable read транзации - 
set transaction isolation level repeatable read;
## в первой сессии добавить новую запись 
insert into persons(first_name, second_name) values('sveta', 'svetova');
postgres=# insert into persons(first_name, second_name) values('sveta', 'svetova');
INSERT 0 1
postgres=*# select id serial, first_name text, second_name text from persons;
 serial |  text  |  text
--------+--------+---------
      1 | ivan   | ivanov
      2 | petr   | petrov
      3 | sergey | sergeev
      4 | sveta  | svetova
(4 rows)
## сделать select* from persons во второй сессии*
postgres=# select id serial, first_name text, second_name text from persons;
 serial |  text  |  text
--------+--------+---------
      1 | ivan   | ivanov
      2 | petr   | petrov
      3 | sergey | sergeev
(3 rows)
## видите ли вы новую запись и если да то почему?
новую запись не видно
## завершить первую транзакцию - commit;
postgres=*# commit;
COMMIT
## сделать select from persons во второй сессии
postgres=# select id serial, first_name text, second_name text from persons;
 serial |  text  |  text
--------+--------+---------
      1 | ivan   | ivanov
      2 | petr   | petrov
      3 | sergey | sergeev
      4 | sveta  | svetova
(4 rows)
## видите ли вы новую запись и если да то почему?
новая запись видна тк применили commit для успешного завершения транзакции
## завершить вторую транзакцию
## сделать select * from persons во второй сессии
postgres=# select id serial, first_name text, second_name text from persons;
 serial |  text  |  text
--------+--------+---------
      1 | ivan   | ivanov
      2 | petr   | petrov
      3 | sergey | sergeev
      4 | sveta  | svetova
(4 rows)
## видите ли вы новую запись и если да то почему?
новая запись видна тк применили commit для успешного завершения транзакции
