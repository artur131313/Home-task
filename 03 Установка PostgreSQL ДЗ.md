<b>ДЗ к занятию 3.Установка PostgreSQL
<b>      Установка и настройка PostgteSQL в контейнере Docker</b>

Цель:
установить PostgreSQL в Docker контейнере
настроить контейнер для внешнего подключения

Описание/Пошаговая инструкция выполнения домашнего задания:
создать ВМ с Ubuntu 20.04/22.04 или развернуть докер любым удобным способом
поставить на нем Docker Engine
сделать каталог /var/lib/postgres
развернуть контейнер с PostgreSQL 15 смонтировав в него /var/lib/postgresql
развернуть контейнер с клиентом postgres
подключится из контейнера с клиентом к контейнеру с сервером и сделать таблицу с парой строк
подключится к контейнеру с сервером с ноутбука/компьютера извне инстансов ЯО/места установки докера
удалить контейнер с сервером
создать его заново
подключится снова из контейнера с клиентом к контейнеру с сервером
проверить, что данные остались на месте
оставляйте в ЛК ДЗ комментарии что и как вы делали и как боролись с проблемами

# Выполнение
Создана ВМ в Яндекс облаке, сгенерирован SSH ключ. Подключение к ВМ через Putty и установка Docker командой:

curl -fsSL https://get.docker.com -o get-docker.sh && sudo sh get-docker.sh && rm get-docker.sh && sudo usermod -aG docker $USER && newgrp docker
![1 установка docker](https://github.com/user-attachments/assets/70a67902-a104-4bb5-a6c0-c2fb7974ac6e)

Далее создал docker-сеть, командой:

sudo docker network create pg-net

67f6271172ef8aafcba1aaa5753776779ef7499b194c8850ade8339726520580
![2 создали docker-сеть](https://github.com/user-attachments/assets/2a18749e-8a05-41ca-8fbb-1d4d8821d517)

Далее подключил созданную сеть к контейнеру сервера Postgres, проверяем, что подключились, командой:

sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15

Проверяем, что подключились через отдельный контейнер:

sudo docker ps -a
![3 подключаем созданную сеть к контейнеру сервера Postgres, проверяем, что подключились](https://github.com/user-attachments/assets/891bc2cf-9963-43e6-925c-ae2185ff81b9)

Далее заходим в Postgres, командой:

sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
![4 заходим в Postgres](https://github.com/user-attachments/assets/d8b79023-1e93-4765-8283-68e8445ba1ce)

Далее создаём БД otus, таблицу test и добавляем 2 записи:
CREATE DATABASE otus; 
![5 Создана БД otus, таблица test и добавлено 2 записи](https://github.com/user-attachments/assets/3334064f-ffc8-452a-9605-cc6317f431f7)

Далее подключились из другой сессии к БД otus, таблица test есть:
![6 Подключились извне сессии к БД otus, таблица test есть](https://github.com/user-attachments/assets/eac51bfb-ced6-41fb-b496-8aa79e7e01fe)

Далее удалили контейнер:

sudo docker stop 681fd7ac4707 

sudo docker rm 681fd7ac4707 
![7 Удалили контейнер](https://github.com/user-attachments/assets/fb7a3f11-81b1-47e3-a574-a30b21c09c79)

Далее создали и запустили новый контейнер из оставшегося образа:

sudo docker run --name pg-server --network pg-net -e POSTGRES_PASSWORD=postgres -d -p 5432:5432 -v /var/lib/postgres:/var/lib/postgresql/data postgres:15

sudo docker run -it --rm --network pg-net --name pg-client postgres:15 psql -h pg-server -U postgres
![8 Создали и запустили новый контейнер из оставшегося образа](https://github.com/user-attachments/assets/43409d07-0465-460e-8f41-e877ef976766)

Проверили ID нового контейнера:
sudo docker ps -a
![9 Проверка ID нового контейнера](https://github.com/user-attachments/assets/6f6f555f-635b-4aa9-8e4b-9d7bdda18db1)

Проверка подлючения к БД через 2ю сессию:
![9_1 Проверка подлючения к БД через 2ю сессию](https://github.com/user-attachments/assets/4bb52249-8a38-4853-88a9-485828b84e09)

Все данные в контейнере остались: база данных и таблица с данными.