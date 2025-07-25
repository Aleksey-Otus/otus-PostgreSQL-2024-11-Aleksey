<br>•	создайте новый кластер PostgresSQL 14
<br>•	зайдите в созданный кластер под пользователем postgres
<br>•	создайте новую базу данных testdb

Создана ВМ в Яндекс облаке, сгенерирован SSH ключ. Подключение к ВМ через Putty и установка на нее PostgreSQL 17 через sudo apt:
sudo apt update && sudo DEBIAN_FRONTEND=noninteractive apt upgrade -y -q && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo DEBIAN_FRONTEND=noninteractive apt -y install postgresql-17
pg_lsclusters

![1](https://github.com/user-attachments/assets/9df8e358-da6a-4632-9a30-420a4a017242)

<br>•	зайдите в созданную базу данных под пользователем postgres
<br>•	создайте новую схему testnm
<br>•	создайте новую таблицу t1 с одной колонкой c1 типа integer
<br>•	вставьте строку со значением c1=1

![2](https://github.com/user-attachments/assets/ac874896-a488-45eb-8c02-0e49fe3ccbbe)

<br>•	создайте новую роль readonly
<br>•	дайте новой роли право на подключение к базе данных testdb
<br>•	дайте новой роли право на использование схемы testnm
<br>•	дайте новой роли право на select для всех таблиц схемы testnm
<br>•	создайте пользователя testread с паролем test123
<br>•	дайте роль readonly пользователю testread

![3](https://github.com/user-attachments/assets/87665fa8-10d3-45c1-b755-f467cb9d8840)

<br>•	зайдите под пользователем testread в базу данных testdb
<br>•	сделайте select * from t1;
<br>•	получилось? (могло если вы делали сами не по шпаргалке и не упустили один существенный момент про который позже)
<br>•	напишите что именно произошло в тексте домашнего задания
<br>•	у вас есть идеи почему? ведь права то дали?
<br>•	посмотрите на список таблиц

Ответ на вопрос: Нет, т.к. нет доступа на select для схемы public (в которой создана таблица t1), дали доступ на схему testnm.
Подключение к базе testdb под пользователем testread производил командой:
sudo -u postgres psql -U testread -h 127.0.0.1 -W -d testdb
пришлось указать, что используется подключение по сети 127.0.0.1, т.к. файл pg_hba.conf не менял.

![4](https://github.com/user-attachments/assets/c9f7e2b0-b12a-401e-8c38-dc04b8a5aa01)

<br>•	вернитесь в базу данных testdb под пользователем postgres
<br>•	удалите таблицу t1
<br>•	создайте ее заново но уже с явным указанием имени схемы testnm
<br>•	вставьте строку со значением c1=1
<br>•	зайдите под пользователем testread в базу данных testdb
<br>•	сделайте select * from testnm.t1;
<br>•	получилось?

Ответ на вопрос: Нет, т.к. доступ на select всех таблиц схемы testnm для роли readonly был дан только для существующих на тот момент времени таблиц, а затем таблицу testnm.t1 пересоздали.

![5](https://github.com/user-attachments/assets/3c8284eb-bfb8-4ca1-aa56-f315bc3e4712)

<br>•	как сделать так чтобы такое больше не повторялось? если нет идей - смотрите шпаргалку
<br>•	сделайте select * from testnm.t1;
<br>•	получилось?

Ответ на вопрос: Да, т.к. дал повторно доступ на select всех таблиц схемы testnm для роли readonly. Командой:
grant SELECT on all TABLEs in SCHEMA testnm TO readonly

![6](https://github.com/user-attachments/assets/2e0f3b94-525b-4145-afe7-03415e9dbdf0)

<br>•	теперь попробуйте выполнить команду create table t2(c1 integer); insert into t2 values (2);
<br>•	а как так? нам же никто прав на создание таблиц и insert в них под ролью readonly?

Ответ на вопрос: Выполнить команду create table t2 не даёт, т.к. права на CREATE TABLE убраны (использую 17 версию Postgres).

![7](https://github.com/user-attachments/assets/c8ab2690-d0bc-493a-8e02-300a697a1a40)

Выдал доступ на создание объектов и таблиц пользователю testread на схему testnm, чтобы мог сам создать таблицы. Использовал команды:
Привилегии CREATE, чтобы пользователь смог создавать новые объекты внутри схемы:
GRANT CREATE ON SCHEMA testnm TO testread;
Выдал пользователю testread доступ на создание таблиц в схеме testnm:
GRANT ALL ON ALL TABLES IN SCHEMA testnm TO testread;
Создал таблицу testnm.t2 под пользователем testread

![8](https://github.com/user-attachments/assets/e3da4be9-d2b3-4a16-9f62-a541fdb08d23)

Создал таблицу testnm.t3 под пользователем testread и вставил записиь в testnm.t1

![9](https://github.com/user-attachments/assets/e75b9756-3673-4846-b7d0-7cd322388392)

