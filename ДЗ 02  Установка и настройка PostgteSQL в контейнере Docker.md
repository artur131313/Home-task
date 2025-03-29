# Практическая работа: Установка и настройка PostgreSQL в контейнере Docker

## 🌟 Цель:
- Установить PostgreSQL в Docker-контейнере.
- Настроить контейнер для внешнего подключения.

---

## 📋 Пошаговая инструкция

### 1. Подготовка ВМ
- Создана виртуальная машина в Яндекс облаке.

### 2. Установил Docker Engine
```bash
curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER && newgrp docker
```
Проверка версии:
```bash
docker --version
```
![image](https://github.com/user-attachments/assets/0acc555f-2550-4c6e-9922-a760cce8e4b8)


---


### 3. Создал docker-сеть.
```bash
sudo docker network create pg-net
```

---

### 4. Подключил созданную сеть к контейнеру сервера Postgres
```bash
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
```
![image](https://github.com/user-attachments/assets/a4a53594-a9c6-49a6-b26f-f9c77020a32e)
---

### 5. Запустил отдельный контейнер с клиентом в общей сети с БД:
```bash
sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
```

![image](https://github.com/user-attachments/assets/f21d9133-544c-4a37-8035-3a1ac10a91a7)

---


### 6. Создал БД otus, таблицу people и добавляем 2 записи:
```sql
CREATE DATABASE otus;

![image](https://github.com/user-attachments/assets/f59c5c74-c602-4723-91d4-b0093df93f9a)

CREATE TABLE people (
  id SERIAL PRIMARY KEY,
  name TEXT
);

INSERT INTO people (name) VALUES ('Alice'), ('Bob');
SELECT * FROM people;
```
![image](https://github.com/user-attachments/assets/480ee378-8b14-4698-894b-1fa17b0a429b)

---

### 7. Подключение извне
- Откройте порт 5432 в настройках фаервола/безопасности.
- На своём ПК установите `psql`, например:

```bash
sudo apt install postgresql-client
```

- Подключитесь:
```bash
psql -h <IP_вашей_ВМ> -U admin -d testdb
```
---
--
![image](https://github.com/user-attachments/assets/82221f81-f0c8-486c-87c4-bc41899a98ec)

### 8. Удаление и повторный запуск контейнера
```bash
docker stop pg-server
docker rm pg-server
```

Повторный запуск:
```bash
docker run -d \
  --name pg-server \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=testdb \
  -v /var/lib/postgres:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:15
```

---

### 9. Повторное подключение из контейнера-клиента
```bash
docker run -it --rm \
  --network host \
  postgres:15 psql -h localhost -U admin -d testdb
```

Проверьте, что данные остались:
```sql
SELECT * FROM people;
```
![image](https://github.com/user-attachments/assets/fa5e425f-6f33-47ca-9bf0-8bf0dc9f3305)

---

## 📝 Комментарии по выполнению

- Docker был установлен вручную через apt.
- Для монтирования данных использовался отдельный каталог `/var/lib/postgres`.
- При подключении из внешнего клиента пришлось открыть порт 5432 в фаерволе и проверить `listen_addresses` в `postgresql.conf` (если нужно).
- После пересоздания контейнера данные остались на месте, так как использовался внешний том.

---

## ✅ Проверка выполненного задания
- [x] ВМ с Docker
- [x] PostgreSQL 15 в контейнере
- [x] Внешний том `/var/lib/postgres`
- [x] Клиент в отдельном контейнере
- [x] Успешное подключение и работа с БД
- [x] Подключение извне
- [x] Пересоздание и проверка сохранности данных
