<br>•	развернуть виртуальную машину любым удобным способом
<br>•	поставить на неё PostgreSQL 15 любым способом
<br>•	настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с надежностью в случае аварийной перезагрузки виртуальной машины
<br>•	нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)
<br>•	написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему

Создана ВМ в Яндекс облаке, сгенерирован SSH ключ. Подключение к ВМ и установка на нее PostgreSQL 17:
sudo apt update && sudo apt upgrade -y && sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list' && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add - && sudo apt-get update && sudo apt-get -y install postgresql
pg_lsclusters

![1](https://github.com/user-attachments/assets/598df783-0510-4dac-84bc-aaf75ae50ef5)

Проверка параметров и наличие утилиты pgbench

![2](https://github.com/user-attachments/assets/e598ef88-7d26-4edc-875d-5ad547da39c3)

Инициализация pgbench

![3](https://github.com/user-attachments/assets/3406f172-461e-4273-8f77-abc5e001bb37)

Запускаем 1-й тест на 30 секунд:
pgbench -U postgres -h 0.0.0.0 -T 30 test
Результат:
latency average = 3.434 ms
initial connection time = 12.509 ms

![4](https://github.com/user-attachments/assets/dba9c22a-2be5-4667-9d6b-19d66e562f0e)

Запускаем 1-й тест на 30 секунд после добавления дополнительных настроек:
pgbench -U postgres -h 0.0.0.0 -T 30 test
Результат:
latency average = 3.483 ms
initial connection time = 12.556 ms
tps = 287.082169 (without initial connection time)
Настройки:
-- Memory Configuration
ALTER SYSTEM SET shared_buffers TO '1GB';
ALTER SYSTEM SET effective_cache_size TO '3GB';
ALTER SYSTEM SET work_mem TO '14MB';
ALTER SYSTEM SET maintenance_work_mem TO '205MB';

-- Checkpoint Related Configuration
ALTER SYSTEM SET min_wal_size TO '2GB';
ALTER SYSTEM SET max_wal_size TO '3GB';
ALTER SYSTEM SET checkpoint_completion_target TO '0.9';
ALTER SYSTEM SET wal_buffers TO '-1';

-- Network Related Configuration
ALTER SYSTEM SET listen_addresses TO '*';
ALTER SYSTEM SET max_connections TO '100';

-- Storage Configuration
ALTER SYSTEM SET random_page_cost TO '1.1';
ALTER SYSTEM SET effective_io_concurrency TO '200';

-- Worker Processes Configuration
ALTER SYSTEM SET max_worker_processes TO '8';
ALTER SYSTEM SET max_parallel_workers_per_gather TO '2';
ALTER SYSTEM SET max_parallel_workers TO '2';

![5](https://github.com/user-attachments/assets/218c6e01-9525-4a16-8f9d-eff96dc0ef3b)

Запускаем повторно 1-й тест после восстановления настроек:
alter system reset all;
pgbench -U postgres -h 0.0.0.0 -T 30 test
Результат:
latency average = 3.844 ms
initial connection time = 12.436 ms
tps = 260.158390 (without initial connection time)

![6](https://github.com/user-attachments/assets/120c8aed-8a71-48cb-ab34-9269bd839957)

Результаты 1-го теста примерно сопоставимы.


Запускаем 2-й расширенный тест с новыми настройками:
pgbench -U postgres -h 0.0.0.0 -c 50 -j 2 -P 10 -T 60 test
Результат:
duration: 60 s
number of transactions actually processed: 17509
number of failed transactions: 0 (0.000%)
latency average = 170.463 ms
latency stddev = 233.724 ms
initial connection time = 519.860 ms
tps = 292.231109 (without initial connection time)
Настройки:
-- Memory Configuration
ALTER SYSTEM SET shared_buffers TO '4GB';
ALTER SYSTEM SET effective_cache_size TO '12GB';
ALTER SYSTEM SET work_mem TO '57MB';
ALTER SYSTEM SET maintenance_work_mem TO '819MB';

-- Checkpoint Related Configuration
ALTER SYSTEM SET min_wal_size TO '2GB';
ALTER SYSTEM SET max_wal_size TO '3GB';
ALTER SYSTEM SET checkpoint_completion_target TO '0.9';
ALTER SYSTEM SET wal_buffers TO '-1';

-- Network Related Configuration
ALTER SYSTEM SET listen_addresses TO '*';
ALTER SYSTEM SET max_connections TO '100';

-- Storage Configuration
ALTER SYSTEM SET random_page_cost TO '1.1';
ALTER SYSTEM SET effective_io_concurrency TO '200';

-- Worker Processes Configuration
ALTER SYSTEM SET max_worker_processes TO '8';
ALTER SYSTEM SET max_parallel_workers_per_gather TO '2';
ALTER SYSTEM SET max_parallel_workers TO '2';

![7](https://github.com/user-attachments/assets/2e00b919-2dc4-4d15-93d6-c95d9bcead84)

Запускаем 2-й расширенный тест со стандартными настройками:
pgbench -U postgres -h 0.0.0.0 -c 50 -j 2 -P 10 -T 60 test
Результат:
duration: 60 s
number of transactions actually processed: 14900
number of failed transactions: 0 (0.000%)
latency average = 200.027 ms
latency stddev = 273.561 ms
initial connection time = 515.139 ms
tps = 249.443649 (without initial connection time)

![8](https://github.com/user-attachments/assets/00e50a99-6f04-4e63-b992-792020319ed5)

Результаты 2-го теста лучше с дополнительными настройкаим.

![9](https://github.com/user-attachments/assets/0bcff13c-2e1f-4f10-984d-6a42276a7552)

Данные ВМ:

![10](https://github.com/user-attachments/assets/1620b98f-2d39-4c4b-b55c-ab8efe042baa)

Дашборд с нагрузками к базе:

![11](https://github.com/user-attachments/assets/d5a1c9b9-4472-4b49-bfe4-b05730284d89)

