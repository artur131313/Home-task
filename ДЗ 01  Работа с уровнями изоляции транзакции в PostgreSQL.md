# Домашнее задание: Работа с уровнями изоляции транзакции в PostgreSQL

## Цель

Научиться управлять уровнем изоляции транзакций в PostgreSQL и понимать особенности работы уровней **READ COMMITTED** и **REPEATABLE READ**.

---

## Пошаговая инструкция выполнения домашнего задания

### 1. Создание среды

- Создайте новый проект в **Яндекс Облаке** или на любой **ВМ/докере**.
- Создайте **инстанс виртуальной машины** с дефолтными параметрами.
- Добавьте свой **SSH-ключ** в **metadata** ВМ.
- Подключитесь по **SSH** (первая сессия). Не забудьте про `ssh-add`.
- Установите **PostgreSQL**.
- Откройте **вторую SSH-сессию**.
- Запустите в обеих сессиях `psql` от пользователя `postgres`.
- Выключите **auto commit** командой:
  ```sql
  \set AUTOCOMMIT off
  ```

---

### 2. Работа с транзакциями

#### **Шаг 1: Создание таблицы и наполнение данными**

В **первой сессии** выполните:

```sql
CREATE TABLE persons(
    id SERIAL,
    first_name TEXT,
    second_name TEXT
);
INSERT INTO persons(first_name, second_name) VALUES('ivan', 'ivanov');
INSERT INTO persons(first_name, second_name) VALUES('petr', 'petrov');
COMMIT;
```

#### **Шаг 2: Проверка уровня изоляции**

В **обеих сессиях** выполните:

```sql
SHOW TRANSACTION ISOLATION LEVEL;
```

Ожидаемый результат: `read committed` (уровень изоляции по умолчанию).
![image](https://github.com/user-attachments/assets/2731d14a-ab9a-4698-90c5-edfb187cd19c)




#### **Шаг 3: Работа с read committed**

1. В **первой сессии** начните транзакцию:
   ```sql
   BEGIN;
   INSERT INTO persons(first_name, second_name) VALUES('sergey', 'sergeev');
   ```
2. В **второй сессии** начните транзакцию и выполните:
   ```sql
   BEGIN;
   SELECT * FROM persons;
   ```
   **Вопрос:** Видите ли вы новую запись? Если да, то почему?
   Ответ: Новой записи не видно
   ![image](https://github.com/user-attachments/assets/71ace946-8b27-4a17-96e3-9ab46ebb45e6)

4. В **первой сессии** завершите транзакцию:
   ```sql
   COMMIT;
   ```
5. В **второй сессии** снова выполните `SELECT * FROM persons;`. **Вопрос:** Видите ли вы новую запись? Если да, то почему?
   Ответ: Да, так как транзакция в первой сессии завершена
![image](https://github.com/user-attachments/assets/0e17c519-647e-47bf-808c-2466a306a761)


7. Завершите транзакцию во **второй сессии**:
   ```sql
   COMMIT;
   ```

#### **Шаг 4: Работа с repeatable read**

1. В **обеих сессиях** установите уровень изоляции:
   ```sql
   SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
   ```
2. В **первой сессии** начните транзакцию и выполните:
   ```sql
   BEGIN;
   INSERT INTO persons(first_name, second_name) VALUES('sveta', 'svetova');
   ```
3. В **второй сессии** выполните:
   ```sql
   BEGIN;
   SELECT * FROM persons;
   ```
   **Вопрос:** Видите ли вы новую запись? Если да, то почему?
   Ответ: Новая запись не видна.
![image](https://github.com/user-attachments/assets/e1ff1358-7bd4-4133-941a-5b69266c3ea9)



5. В **первой сессии** завершите транзакцию:
   ```sql
   COMMIT;
   ```
6. В **второй сессии** снова выполните `SELECT * FROM persons;`. **Вопрос:** Видите ли вы новую запись? Если да, то почему?
Ответ: Вижу потому, что  транзакция выполнилась и данные
![image](https://github.com/user-attachments/assets/823cfbcc-65b1-45d6-8ce3-894ca957ad08)





7. Завершите транзакцию во **второй сессии**:
   ```sql
   COMMIT;
   ```
8. Выполните `SELECT * FROM persons;` во **второй сессии**. **Вопрос:** Видите ли вы новую запись? Если да, то почему?
![image](https://github.com/user-attachments/assets/74a073ef-fec1-489f-a796-fd090eec8d51)


---

## Итоги

- **READ COMMITTED** позволяет видеть данные других транзакций после их фиксации.
- **REPEATABLE READ** фиксирует "снимок" данных на момент начала транзакции, игнорируя последующие изменения до её завершения.
- Разница между уровнями изоляции важна для обеспечения целостности данных в многопоточных системах.

