# Практическая работа: Установка и настройка PostgreSQL в контейнере Docker

## 🌟 Цель:
- Установить PostgreSQL в Docker-контейнере.
- Настроить контейнер для внешнего подключения.

---

## 📋 Пошаговая инструкция

### 1. Подготовка ВМ
- Создайте виртуальную машину с Ubuntu 20.04 или 22.04, либо установите Docker любым удобным способом.

### 2. Установка Docker Engine
```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

Проверьте версию:
```bash
docker --version
```
![image](https://github.com/user-attachments/assets/0acc555f-2550-4c6e-9922-a760cce8e4b8)


Добавьте текущего пользователя в группу `docker`:
```bash
sudo usermod -aG docker $USER
# Перезапустите сессию или выполните:
newgrp docker
```

---
![image](https://github.com/user-attachments/assets/a4a53594-a9c6-49a6-b26f-f9c77020a32e)

### 3. Создание каталога для данных PostgreSQL
```bash
sudo mkdir -p /var/lib/postgres
sudo chown 999:999 /var/lib/postgres
```

---

### 4. Запуск контейнера с PostgreSQL 15
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

### 5. Запуск контейнера с клиентом PostgreSQL
```bash
docker run -it --rm \
  --network host \
  postgres:15 psql -h localhost -U admin -d testdb
```

При появлении запроса пароля введите `secret`.
![image](https://github.com/user-attachments/assets/f21d9133-544c-4a37-8035-3a1ac10a91a7)

---
![image](https://github.com/user-attachments/assets/f59c5c74-c602-4723-91d4-b0093df93f9a)

### 6. Создание таблицы и добавление строк
```sql
CREATE TABLE people (
  id SERIAL PRIMARY KEY,
  name TEXT
);

INSERT INTO people (name) VALUES ('Alice'), ('Bob');
SELECT * FROM people;
```

---
![image](https://github.com/user-attachments/assets/82221f81-f0c8-486c-87c4-bc41899a98ec)

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
Пароль: `secret`

---

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
