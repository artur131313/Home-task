# Домашнее задание  
## Тема: Работа с join'ами, статистикой

---

### 🎯 Цель:
- Знать и уметь применять различные виды join'ов  
- Строить и анализировать план выполнения запроса  
- Оптимизировать запрос  
- Уметь собирать и анализировать статистику для таблицы  

---

## 📌 Описание
В результате выполнения ДЗ вы научитесь пользоваться различными вариантами соединения таблиц.

Навыки, которые тренируются:
- написание запросов с различными типами соединений  

---

## 📂 Структура таблиц

### Таблица `employees`
| id | name     | department_id |
|----|----------|---------------|
| 1  | Alice    | 10            |
| 2  | Bob      | 20            |
| 3  | Charlie  | null          |
| 4  | Diana    | 10            |

### Таблица `departments`
| id | department_name |
|----|-----------------|
| 10 | IT              |
| 20 | HR              |
| 30 | Finance         |

### Таблица `projects`
| id | project_name | employee_id |
|----|--------------|-------------|
| 1  | Apollo       | 1           |
| 2  | Artemis      | 3           |
| 3  | Hermes       | null        |

---

## 🧩 Прямое соединение (INNER JOIN)

```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

> 💬 Комментарий:  
> Показывает только тех сотрудников, у которых указан департамент, и он существует в таблице `departments`.
![image](https://github.com/user-attachments/assets/aba3373a-631c-4ad9-aa32-19ef2681d93b)


---

## 🧩 Левостороннее соединение (LEFT JOIN)

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

> 💬 Комментарий:  
> Показывает всех сотрудников, даже если у них не указан департамент (`department_id IS NULL`) или он отсутствует в `departments`.
![image](https://github.com/user-attachments/assets/4ab4304b-cd4d-4773-93ef-6f2004b41ce9)

---

## 🧩 Правостороннее соединение (RIGHT JOIN)

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

> 💬 Комментарий:  
> Показывает все департаменты, даже если в них нет сотрудников.


![image](https://github.com/user-attachments/assets/961ccdd2-7ae7-4203-b63b-6ee6cfe892a3)

---

## 🧩 Кросс соединение (CROSS JOIN)

```sql
SELECT e.name, p.project_name
FROM employees e
CROSS JOIN projects p;
```

> 💬 Комментарий:  
> Формирует декартово произведение — каждая строка из `employees` комбинируется с каждой строкой из `projects`.
![image](https://github.com/user-attachments/assets/1e7fe2a5-1428-4c20-a4da-26fc8cba8cb7)

---

## 🧩 Полное соединение (FULL OUTER JOIN)

```sql
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

> 💬 Комментарий:  
> Возвращает все записи из обеих таблиц, независимо от наличия соответствия.

![image](https://github.com/user-attachments/assets/c02015fb-a2c6-47de-8264-e0673d9e56ef)

---

## 🧩 Комбинированный запрос с несколькими типами соединений

```sql
SELECT e.name, d.department_name, p.project_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
LEFT JOIN projects p ON e.id = p.employee_id;
```

> 💬 Комментарий:  
> Показывает всех сотрудников с их департаментами и проектами. Даже если сотрудник не участвует в проекте — он будет в результате.

![image](https://github.com/user-attachments/assets/ddccfee5-aacb-4baf-9d62-7916fbc7a91a)


---

