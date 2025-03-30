# Практическая работа: Установка и настройка PostgreSQL

## Цель

1. Создать дополнительный диск для уже существующей виртуальной машины, размечать его и создать на нём файловую систему.  
2. Перенести содержимое базы данных PostgreSQL на дополнительный диск.  
3. Перенести содержимое БД PostgreSQL между виртуальными машинами.

---

## Описание / Пошаговая инструкция выполнения

### 1. Создание ВМ и установка PostgreSQL

- Создал виртуальную машину с Ubuntu 20.04 в Яндекс Облаке.
- Установил PostgreSQL 15:

```bash
sudo apt update
sudo apt-get -y install postgresql
```

- Проверил статус кластера:

```bash
sudo -u postgres pg_lsclusters
```

- Вошел под пользователем `postgres` и создал тестовую таблицу:

```bash
sudo -u postgres psql
```

```sql
create table test(c1 text);
insert into test values('1');
\q
```

---

### 2. Остановил кластера PostgreSQL

```bash
sudo -u postgres pg_ctlcluster 15 main stop
```

---

### 3. Добавление и инициализация дополнительного диска

- Создайте диск размером 10GB.
- Присоедините его к виртуальной машине.
- Найдите имя диска (например `/dev/sdb`):

```bash
lsblk
```

- Разметьте и создайте файловую систему:

```bash
sudo fdisk /dev/sdb
# внутри fdisk: n -> p -> <Enter> -> <Enter> -> w
sudo mkfs.ext4 /dev/sdb1
```

- Создайте точку монтирования и смонтируйте диск:

```bash
sudo mkdir /mnt/data
sudo mount /dev/sdb1 /mnt/data
```

- Убедитесь, что диск смонтирован:

```bash
df -h
```

- Добавьте диск в fstab для автоподключения:

```bash
sudo blkid /dev/sdb1
sudo nano /etc/fstab
# Добавьте строку:
# UUID=<ваш-uuid> /mnt/data ext4 defaults 0 2
```

- Перезагрузите систему и проверьте:

```bash
sudo reboot
df -h
```

---

### 4. Перемещение данных PostgreSQL

- Сделайте `postgres` владельцем новой директории:

```bash
sudo chown -R postgres:postgres /mnt/data
```

- Переместите данные PostgreSQL:

```bash
sudo mv /var/lib/postgresql/15 /mnt/data
```

---

### 5. Попытка запуска кластера

```bash
sudo -u postgres pg_ctlcluster 15 main start
```

> **Результат:** скорее всего не получится — путь к данным прописан в конфигурации.

---

### 6. Обновление конфигурации

- Откройте файл конфигурации:

```bash
sudo nano /etc/postgresql/15/main/postgresql.conf
```

- Измените параметр:

```conf
data_directory = '/mnt/data/15/main'
```

---

### 7. Повторная попытка запуска

```bash
sudo -u postgres pg_ctlcluster 15 main start
```

> **Результат:** если всё настроено правильно — PostgreSQL успешно стартует.

---

### 8. Проверка таблицы

```bash
sudo -u postgres psql
select * from test;
\q
```

---

## ⭐ Задание со звёздочкой

1. Создайте новую ВМ с Ubuntu.
2. Установите PostgreSQL 15.
3. Удалите стандартный каталог данных:

```bash
sudo systemctl stop postgresql
sudo rm -rf /var/lib/postgresql/15/main
```

4. Перемонтируйте внешний диск:

```bash
sudo mkdir /mnt/data
sudo mount /dev/sdb1 /mnt/data
```

5. Измените владельца и конфигурацию:

```bash
sudo chown -R postgres:postgres /mnt/data
sudo nano /etc/postgresql/15/main/postgresql.conf
# data_directory = '/mnt/data/15/main'
```

6. Запустите PostgreSQL:

```bash
sudo systemctl start postgresql
```

7. Проверьте таблицу:

```bash
sudo -u postgres psql
select * from test;
\q
```

---

## Результат

Если всё сделано правильно, PostgreSQL на второй ВМ будет использовать данные с внешнего диска, созданные на первой ВМ.
