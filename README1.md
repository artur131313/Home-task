<b>ДЗ к занятию 3.Установка PostgreSQL
<b>      Установка и настройка PostgteSQL в контейнере Docker</b>

Цель:
установить PostgreSQL в Docker контейнере
настроить контейнер для внешнего подключения


# Установка PostgreSQL в Docker-контейнере и настройка внешнего подключения

## 📌 1. Установка Docker (если не установлен)
```bash
sudo apt update
sudo apt install docker.io -y
```
Проверим, что Docker установлен:
```bash
docker --version
```

---

## 🚀 2. Запуск контейнера с PostgreSQL
```bash
docker run -d \
  --name postgres_container \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydatabase \
  -p 5432:5432 \
  postgres
```

---

## 🔍 3. Проверка запущенного контейнера
```bash
docker ps
```

---

## 🔗 4. Подключение к PostgreSQL внутри контейнера
```bash
docker exec -it postgres_container psql -U myuser -d mydatabase
```

---

## ⚙️ 5. Настройка внешнего подключения
### 5.1. Редактируем **postgresql.conf** для разрешения подключений
```bash
docker exec -it postgres_container bash -c "echo 'listen_addresses = "*"' >> /var/lib/postgresql/data/postgresql.conf"
```

### 5.2. Редактируем **pg_hba.conf**, чтобы разрешить внешние подключения
```bash
docker exec -it postgres_container bash -c "echo 'host all all 0.0.0.0/0 md5' >> /var/lib/postgresql/data/pg_hba.conf"
```

### 5.3. Перезапускаем контейнер
```bash
docker restart postgres_container
```

---

## 🌍 6. Подключение из внешнего клиента
🔹 На хосте:
```bash
psql -h localhost -U myuser -d mydatabase
```

🔹 Если контейнер запущен на удалённом сервере, укажи IP:
```bash
psql -h <server-ip> -U myuser -d mydatabase
```

✅ **Готово!** 🚀
