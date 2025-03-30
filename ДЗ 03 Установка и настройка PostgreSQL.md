# Практическая работа: Установка и настройка PostgreSQL

## Цель

1. Создать дополнительный диск для уже существующей виртуальной машины, размечать его и создать на нём файловую систему.  
2. Перенести содержимое базы данных PostgreSQL на дополнительный диск.  


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
![image](https://github.com/user-attachments/assets/d81b41fc-c549-409d-9d7a-c96ba7ddab47)


- Вошел под пользователем `postgres` и создал тестовую таблицу:

```bash
sudo -u postgres psql
```

```sql
create table test(c1 text);
insert into test values('1');
\q
```
![image](https://github.com/user-attachments/assets/11c6c5ea-9b96-4e16-9aed-80fd7afe0753)

---

### 2. Остановил кластер PostgreSQL

```bash
sudo -u postgres pg_ctlcluster 16 main stop
```

---
![image](https://github.com/user-attachments/assets/517f4f76-8e05-4a18-b59b-cd9ee376e320)

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
UUID="840d0f89-b148-4d78-bfc9-e482ec87117d" /mnt/data ext4 defaults 0 2
```

- Перезагрузил систему и проверил:

```bash
sudo reboot
df -h
```

---
![image](https://github.com/user-attachments/assets/4bed57ea-a0f0-49fa-bc3b-1fbd8ac77334)



### 4. Перемещение данных PostgreSQL

- Сделайте `postgres` владельцем новой директории:

```bash
sudo chown -R postgres:postgres /mnt/data
```

- Переместите данные PostgreSQL:

```bash
sudo mv /var/lib/postgresql/16 /mnt/data


```

---

### 5. Попытка запуска кластера

```bash
sudo -u postgres pg_ctlcluster 16 main start
```

> **Результат:** Error: /var/lib/postgresql/16/main is not accessible or does not exist.

---

### 6. Обновление конфигурации

- Открыл файл конфигурации:

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```

- Изменил параметр:

```conf
data_directory = '/mnt/data/16/main'
```

---

### 7. Повторная попытка запуска

```bash
sudo -u postgres pg_ctlcluster 16 main start
```

> **Результат:**  PostgreSQL успешно стартует.

---

### 8. Проверка таблицы

```bash
sudo -u postgres psql
select * from test;
\q
```

---
![image](https://github.com/user-attachments/assets/817e17c8-0303-4207-99fe-f73044e0a0d8)

Перемещение данных PostgreSQL прошло успешно. Мы видим данные.
