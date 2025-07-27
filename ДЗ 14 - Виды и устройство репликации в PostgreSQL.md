<br>•	На 1 ВМ создаем таблицы test для записи, test2 для запросов на чтение.
<br>•	Создаем публикацию таблицы test и подписываемся на публикацию таблицы test2 с ВМ №2.
<br>•	На 2 ВМ создаем таблицы test2 для записи, test для запросов на чтение.
<br>•	Создаем публикацию таблицы test2 и подписываемся на публикацию таблицы test1 с ВМ №1.
<br>•	3 ВМ использовать как реплику для чтения и бэкапов (подписаться на таблицы из ВМ №1 и №2 ).

В Яндекс облаке созданы 3 ВМ:

![1](https://github.com/user-attachments/assets/82f87a48-40e8-461d-806d-55b35525a524)

IP адреса ВМ:

ВМ1 - 130.193.45.37
ВМ2 - 84.201.151.89
ВМ3 - 158.160.158.244

На всех 3х ВМ установлен Postgresql:

sudo apt-get update
sudo apt-get -y install postgresql

Для доступа к ВМ отредактированны файлы postgresql.conf:

sudo nano /etc/postgresql/16/main/postgresql.conf
#listen_addresses = 'localhost'
listen_addresses = '*'

И pg_hba.conf:

sudo nano /etc/postgresql/16/main/pg_hba.conf
#host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             0.0.0.0/0               md5

На ВМ 1 созданы таблицы test для записи и test2 для запросов на чтение.

create table test as 
select 
  generate_series(1,10) as id,
  md5(random()::text)::char(10) as fio;

create table test2 (id int, fio text);

Создана публикация таблицы test:

CREATE PUBLICATION test_pub FOR TABLE test;

На ВМ 2 созданы таблицы test2 для записи и test для запросов на чтение.

create table test2 as 
select 
  generate_series(1,10) as id,
  md5(random()::text)::char(10) as fio;

create table test (id int, fio text);

Создана публикация таблицы test2:

CREATE PUBLICATION test_pub FOR TABLE test2;

На ВМ 2 создана подписка на ВМ 1:

CREATE SUBSCRIPTION test_sub2 
CONNECTION 'host=130.193.45.37 port=5432 user=postgres password=postgres dbname=test_repl' 
PUBLICATION test_pub WITH (copy_data = true);

![2](https://github.com/user-attachments/assets/4f5e86ff-b4bc-439c-bbcd-d3b06ec65cab)

На ВМ 1 создана подписка на ВМ 2:

CREATE SUBSCRIPTION test_sub1 
CONNECTION 'host=84.201.151.89 port=5432 user=postgres password=postgres dbname=test_repl2' 
PUBLICATION test_pub WITH (copy_data = true);

![3](https://github.com/user-attachments/assets/59964bdc-7f7e-4151-b40d-cddd6984c2d7)

На ВМ 3 созданы две таблицы:

create table test (id int, fio text);
create table test2 (id int, fio text);

Далее на ВМ 3 созданы две подписки на ВМ 1 и на ВМ 2:

CREATE SUBSCRIPTION test_sub3_2
CONNECTION 'host=84.201.151.89 port=5432 user=postgres password=postgres dbname=test_repl2' 
PUBLICATION test_pub WITH (copy_data = true);

CREATE SUBSCRIPTION test_sub3_1 
CONNECTION 'host=130.193.45.37 port=5432 user=postgres password=postgres dbname=test_repl' 
PUBLICATION test_pub WITH (copy_data = true);

![4](https://github.com/user-attachments/assets/f7885059-691b-47d7-b7c9-fa53c9bb2f8f)

На ВМ 3 получены данные из таблицы test с ВМ 1 и из таблицы test2 с ВМ 2

![5](https://github.com/user-attachments/assets/fe05c154-aae0-44ba-a3b5-28e81bcae920)

