Вот оформление домашнего задания в формате Markdown:

```markdown
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

---

## 🧩 Левостороннее соединение (LEFT JOIN)

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

> 💬 Комментарий:  
> Показывает всех сотрудников, даже если у них не указан департамент (`department_id IS NULL`) или он отсутствует в `departments`.

---

## 🧩 Правостороннее соединение (RIGHT JOIN)

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

> 💬 Комментарий:  
> Показывает все департаменты, даже если в них нет сотрудников.

---

## 🧩 Кросс соединение (CROSS JOIN)

```sql
SELECT e.name, p.project_name
FROM employees e
CROSS JOIN projects p;
```

> 💬 Комментарий:  
> Формирует декартово произведение — каждая строка из `employees` комбинируется с каждой строкой из `projects`.

---

## 🧩 Полное соединение (FULL OUTER JOIN)

```sql
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d ON e.department_id = d.id;
```

> 💬 Комментарий:  
> Возвращает все записи из обеих таблиц, независимо от наличия соответствия.

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

---

## 📊 Анализ статистики (для PostgreSQL)

```sql
-- Сбор статистики
ANALYZE employees;
ANALYZE departments;
ANALYZE projects;

-- Просмотр статистики
SELECT * FROM pg_stat_user_tables WHERE relname IN ('employees', 'departments', 'projects');
```

> 💬 Комментарий:  
> Используется для актуализации информации о распределении данных в таблицах. Это важно для построения оптимальных планов выполнения.

---

## 📈 План выполнения запроса (EXPLAIN)

```sql
EXPLAIN ANALYZE
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

> 💬 Комментарий:  
> Показывает, какие индексы и методы доступа используются. Позволяет оценить эффективность запроса.

---

Готово ✅  
```

Если ты работаешь с реальной БД, могу помочь создать скрипт для генерации этих таблиц и наполнения их тестовыми данными.