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


### 6. Создал таблицу people и добавил 2 записи:

```sql
CREATE TABLE people (
  id SERIAL PRIMARY KEY,
  name TEXT
);

INSERT INTO people (name) VALUES ('Alice'), ('Bob');
SELECT * FROM people;
```
![image](https://github.com/user-attachments/assets/480ee378-8b14-4698-894b-1fa17b0a429b)

---

### 7. Подключение извне. Подключился с другого компьютера, таблица people есть:

```bash
psql -p 5432 -U postgres -h 158.160.135.184 -d postgres -W
select * from people;
```
---

![image](https://github.com/user-attachments/assets/c7dd275c-2c01-42f9-b936-b1fe1e6cd931)


![image](https://github.com/user-attachments/assets/82221f81-f0c8-486c-87c4-bc41899a98ec)

### 8. Удаление и повторный запуск контейнера и проверка что данные остались.

Удаление:
```bash
sudo docker stop e85eba8898fb
sudo docker rm e85eba8898fb
```
![image](https://github.com/user-attachments/assets/ff66e14b-d71c-433b-aa8a-f9921c6f907b)


Повторный запуск, подключение из контейнера-клиента и проверка что данные остались:
```bash
sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15
sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
select * from people;
```

---

![image](https://github.com/user-attachments/assets/5781d57c-e293-42d6-aa9d-5709bea35e98)


---

