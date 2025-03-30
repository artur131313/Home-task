# Практическая работа: Установка и настройка PostgreSQL

## Цель

1. Создать дополнительный диск для уже существующей виртуальной машины, размечать его и создать на нём файловую систему.  
2. Перенести содержимое базы данных PostgreSQL на дополнительный диск.  
3. Перенести содержимое БД PostgreSQL между виртуальными машинами.

---

## Описание / Пошаговая инструкция выполнения

### 1. Создание ВМ и установка PostgreSQL

- Создал виртуальную машину с Ubuntu 22.04 в Яндекс Облаке.
- Установил PostgreSQL 16:

```bash
sudo apt update
sudo apt-get -y install postgresql
```

- Проверил статус кластера:

```bash
sudo -u postgres pg_lsclusters
```
![image](https://github.com/user-attachments/assets/da99ced1-c875-4ef1-bf4a-642411154b2c)

- Вошел под пользователем `postgres` и создал тестовую таблицу:

```bash
sudo -u postgres psql
```

```sql
create table test(c1 text);
insert into test values('1');
\q
```
![image](https://github.com/user-attachments/assets/3463f73b-0958-4b2b-997b-42eb698267ae)

---

### 2. Остановил кластер PostgreSQL

```bash
sudo -u postgres pg_ctlcluster 16 main stop
```

---
![image](https://github.com/user-attachments/assets/579e8806-ed72-4830-a25d-3824ef8eaac9)


### 3. Добавил и инициализировал дополнительный диск

- Создал диск размером 10GB.
- Присоединил его к виртуальной машине.
- Найдите имя диска (например `/dev/vdb`):

```bash
lsblk
```
![image](https://github.com/user-attachments/assets/821f8a02-bf0b-49e0-88a5-6e0f883cf70a)

- Разметил и создал файловую систему:

```bash
sudo fdisk /dev/vdb
# внутри fdisk: n -> p -> <Enter> -> <Enter> -> w
sudo mkfs.ext4 /dev/vdb1
```

- Создал точку монтирования и смонтировал диск:

```bash
sudo mkdir /mnt/data
sudo mount /dev/vdb1 /mnt/data
```

- Убедился, что диск смонтирован:

```bash
df -h
```
![image](https://github.com/user-attachments/assets/55b4117a-bdba-4162-8b5c-4455d91bcd55)

- Добавил диск в fstab для автоподключения:

```bash
sudo blkid /dev/vdb1
sudo nano /etc/fstab
# Добавьте строку:
UUID="ec0e1116-99a0-4849-9e6a-04549b80b3ee" /mnt/data ext4 defaults 0 2
```

- Перезагрузите систему и проверьте:

```bash
sudo reboot
df -h
```

---
![image](https://github.com/user-attachments/assets/01bf6a7c-f71a-4153-bd7f-18061fd05487)


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
