<br><b> Выполнение</b>

<br>• Подключились к ВМ с Postgresql, созданной в VirtualBox:

<br>• Проверка:
<br>pgbench --version

<img width="1588" height="688" alt="1" src="https://github.com/user-attachments/assets/ad61366f-8a57-4c73-baef-9f5901c34822" />

<br>• Инициализация теста pgbench:
<br>pgbench -U postgres -h 0.0.0.0 -i TestVacuum

<img width="1771" height="731" alt="2" src="https://github.com/user-attachments/assets/b5d08c7d-882f-4061-823d-3c43da62d5fc" />

<br>• Запуск:
<br>pgbench -c8 -P 6 -T 60 -U postgres TestVacuum

<img width="1537" height="865" alt="3" src="https://github.com/user-attachments/assets/a74a4277-90fc-4f40-9d15-f741e8cf3777" />

<br>• Изменил настройки в файле postgresql.conf:
<br>sudo nano /etc/postgresql/16/main/postgresql.conf

<img width="1718" height="1348" alt="4" src="https://github.com/user-attachments/assets/8e11d623-6b0c-441e-b7f4-86f798881210" />

<br>• Проверка применения настроек

<img width="1738" height="872" alt="5" src="https://github.com/user-attachments/assets/92c424a3-b568-44b2-817a-dbc1af438437" />

<br>• Повторный запуск теста после смены настроек:
<br>pgbench -c8 -P 6 -T 60 -U postgres TestVacuum

<img width="1745" height="996" alt="6" src="https://github.com/user-attachments/assets/59a98dba-8ea0-4582-8d5e-3903221517a6" />

<br> Особых изменений не произошло. Возможно из-за того, что в базе нет тяжёлых объектов.
<br>Было:
<br>latency average = 35.061 ms
<br>latency stddev = 22.617 ms
<br>initial connection time = 139.067 ms
<br>tps = 227.850168 (without initial connection time)

<br>Стало:
<br>latency average = 34.900 ms
<br>latency stddev = 22.020 ms
<br>initial connection time = 146.991 ms
<br>tps = 228.889880 (without initial connection time)


<br>• Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1млн строк
<br>Посмотреть размер файла с таблицей
<br>5 раз обновить все строчки и добавить к каждой строчке любой символ
<br>Посмотреть количество мертвых строчек в таблице и когда последний раз приходил автовакуум
<br>Подождать некоторое время, проверяя, пришел ли автовакуум

<img width="1754" height="927" alt="7" src="https://github.com/user-attachments/assets/ce81ae65-690d-40bb-81f6-2ab11aca346f" />

<br>• После того как пришел автовакуум, мёртвых строчек не осталось

<img width="1775" height="970" alt="8" src="https://github.com/user-attachments/assets/bb0fe8aa-c3e0-4323-a95b-f80acaa67ac9" />

<br>• 5 раз обновить все строчки и добавить к каждой строчке любой символ
<br>Посмотреть размер файла с таблицей
<br>Размер файла таблицы увеличился с 35МБ до 207МБ

<img width="1771" height="950" alt="9" src="https://github.com/user-attachments/assets/e9473c1d-e735-4516-900d-702a1732adba" />

<br>• После выполнения vacuum full. Мёртвые строки удалились и размер файла таблицы стал снова 35МБ
<br>vacuum full test;

<img width="1769" height="1033" alt="10" src="https://github.com/user-attachments/assets/e1c55f08-49ab-43b4-b56e-e98e5a8371a0" />

<br>• Отключить Автовакуум на конкретной таблице
<br>10 раз обновить все строчки и добавить к каждой строчке любой символ
<br>Посмотреть размер файла с таблицей
<br>Объясните полученный результат
<br>Не забудьте включить автовакуум)

<br>• Выполнение:</b>
<br>Автовакуум отключен на новыой таблице vtest и произведено заполнение таблицы значениями
<br>Размер файла таблицы: 64МБ
<br>Затем 10 раз обновлены все строчки таблицы и добавлены к каждой строчке символы '_a'
<br>Размер файла таблицы вырос с 64МБ до 448МБ. Произошло из-за того, что не произведено удаление мёртвых строчек таблицы и они продолжают занимать место на диске

<img width="1766" height="960" alt="11" src="https://github.com/user-attachments/assets/bd53b7ac-b5aa-4292-8fed-3e0c4edc3f9b" />

<br>• После включения автовакуум-а на таблице vtest:
<br>ALTER TABLE vtest SET (autovacuum_enabled = on);
<br>vacuum full vtest;
<br>Размер файла таблицы стал прежним: 64МБ, т.к. из таблицы прошло удаление мёртвых строчек.

<img width="1770" height="786" alt="12" src="https://github.com/user-attachments/assets/66ed5954-52ed-4b15-88b2-870067f63acb" />

