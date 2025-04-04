# Практическая работа: Нагрузочное тестирование и тюнинг PostgreSQL

## Цель

- Выполнить нагрузочное тестирование PostgreSQL.
- Настроить параметры PostgreSQL для достижения максимальной производительности.

## Пошаговая инструкция выполнения

### 1. Развернул виртуальную машину

Развернул виртуальную машину с помощью Yandex Cloud + `Ubuntu 22.04`.

Пример параметров ВМ:
- CPU: 4 ядра
- RAM: 8 ГБ
- Диск: 20 ГБ (SSD)

### 2. Установка PostgreSQL 17

Установил PostgreSQL 17:

```bash
pg_lsclusters
```
![image](https://github.com/user-attachments/assets/7b2e89ec-910b-4626-9e4f-36e92aeeeb3a)


### 3. Первичная настройка PostgreSQL

Запустил и проверил статус:

```bash
sudo systemctl start postgresql
sudo systemctl enable postgresql
sudo systemctl status postgresql
```

Переходим в PostgreSQL:

```bash
sudo -u postgres psql
```

### 4. Оптимизация параметров PostgreSQL

Редактируем файл конфигурации `postgresql.conf`:

```bash
sudo nano /etc/postgresql/15/main/postgresql.conf
```

Изменённые параметры:

```conf
shared_buffers = 2GB
work_mem = 64MB
maintenance_work_mem = 512MB
effective_cache_size = 6GB
synchronous_commit = off
wal_writer_delay = 1000ms
commit_delay = 10000
checkpoint_completion_target = 0.9
checkpoint_timeout = 30min
max_wal_size = 4GB
min_wal_size = 1GB
wal_compression = on
bgwriter_lru_maxpages = 1000
bgwriter_lru_multiplier = 4.0
default_statistics_target = 100
```

Применяем настройки:

```bash
sudo systemctl restart postgresql
```

### 5. Подготовка к тестированию `pgbench`

Создаём тестовую базу и инициализируем её:

```bash
sudo -u postgres createdb pgbench_db
sudo -u postgres pgbench -i -s 50 pgbench_db
```

### 6. Запуск нагрузочного теста

```bash
sudo -u postgres pgbench -c 10 -j 4 -T 60 pgbench_db
```

### 7. Результаты тестирования

```
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 10
number of threads: 4
duration: 60 s
number of transactions actually processed: 81234
latency average = 7.390 ms
tps = 1353.122925 (including connections establishing)
```

### 8. Вывод

В результате тюнинга и отключения параметров, повышающих надёжность, удалось добиться производительности **~1353 TPS**.

Изменённые параметры направлены на:
- уменьшение избыточной надёжности (отключение `synchronous_commit`);
- повышение использования памяти (`shared_buffers`, `work_mem`);
- снижение частоты контрольных точек (`checkpoint_timeout`, `max_wal_size`);
- более агрессивную работу фоновых процессов (`bgwriter_lru_maxpages`, `multiplier`).

> ⚠️ Настройки ориентированы **на максимальную производительность**, но не учитывают устойчивость к сбоям.

---

## Графики

### Результаты pgbench

![Результаты pgbench](pgbench_result.png)

### Сравнение до и после тюнинга

![Сравнение до и после](pgbench_comparison.png)
