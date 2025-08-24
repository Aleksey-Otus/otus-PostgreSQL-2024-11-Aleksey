<br><b> Выполнение

<br>• Создана ВМ в Яндекс облаке, сгенерирован SSH ключ.

<br>• Установка Postgresql:

sudo apt-get -y install postgresql

<br>• Зашли из под пользователя postgres в psql:

sudo -u postgres psql

<br>• Создал таблицу и заполнил тестовым значением:

create table test (c1 text);

insert into test values('1');

<br>• Просмотрел файл postgresql.conf:

sudo nano /etc/postgresql/16/main/postgresql.conf

<br>• Проверил куда ссылается параметр data_directory:

data_directory = '/var/lib/postgresql/16/main'

<br>• При создании ВМ сразу указал 2 диска:

Новый диск: vdb

<br><b>Далее из замечаний по ДЗ:</b>

<br>• добавляем диск и смотрим, есть ли он

sudo parted -l | grep Error - должна появиться ошибка с нераспознанной меткой диска

<br>• проверяем, что диск виден, но не размечен

lsblk

<br>• форматируем и размечаем диск

sudo parted /dev/vdb mklabel gpt
sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
sudo mkfs.ext4 -L datapartition /dev/vdb1

<br>• проверяем, что раздел создался и с диском все ок:

sudo lsblk -o NAME,FSTYPE,LABEL,UUID | grep -v loop

<br>• создаем на диске папку и монтируем этот диск

sudo mkdir -p /mnt/data
sudo mount -a

<br>• перезагружаемся

sync; sudo reboot

<br>• после перезагрузки проверяем, что с диском все еще все ок:

df -h /mnt/data
<br>• выдаем права пользователю postgres

sudo chown -R postgres:postgres /mnt/data/

<br>• перемещаем все в /mnt/data

sudo mv /var/lib/postgresql/16 /mnt/data

<br>• редактируем postgresql.conf

sudo nano /etc/postgresql/16/main/postgresql.conf

<br>• запускаем сервис postgresql и сам кластер:

sudo systemctl start postgresql.service
sudo -u postgres pg_ctlcluster 16 main start

<br>• Подключаемся к БД и проверяем данные.

<br><b>Всё хорошо. Таблица и данные присутствуют:</b>

<img width="1777" height="1408" alt="1" src="https://github.com/user-attachments/assets/59496210-e971-4c08-8136-958d08605f1d" />


<img width="1775" height="1436" alt="2" src="https://github.com/user-attachments/assets/dde97598-4720-4983-ad32-95506f2cfbc5" />


<img width="1773" height="623" alt="3" src="https://github.com/user-attachments/assets/76b92e8d-73c0-4e47-ad2e-627ed4650525" />




