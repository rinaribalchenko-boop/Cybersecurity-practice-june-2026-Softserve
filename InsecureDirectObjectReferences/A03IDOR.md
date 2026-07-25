# Звіт з виконання практичної роботи: A03 Injection (SQL Injection intro)
## Мета
Ознайомитись на практиці з основними концепціями баз даних (DML, DDL, DCL), а також дослідити механізми виникнення та експлуатації вразливостей типу SQL-ін'єкція на навчальній платформі WebGoat. Навчитися компрометувати конфіденційність, цілісність та доступність даних (тріада CIA) за допомогою вразливих полів вводу.

---

## Хід виконання
Всі завдання я виконувала у віртуальному середовищі Kali Linux, використовуючи розгорнутий контейнер WebGoat.

### Частина 1: Основи SQL запитів
Перед тим як переходити до ін'єкцій, я виконала кілька базових завдань для розуміння синтаксису SQL.

1. Вибірка даних (Data Manipulation Language - DML)
Завдання полягало у отриманні відділу, де працює співробітник Bob Franco. Я сформувала наступний запит:
```bash
SELECT department FROM employees WHERE first_name = 'Bob' AND last_name = 'Franco';
```
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_40_05" src="https://github.com/user-attachments/assets/6947e353-30a8-4879-aedf-eb2d9a31c75a" />

### 2. Оновлення даних (DML)
Далі необхідно було змінити відділ співробітника Tobi Barnett на 'Sales'. Для цього я використала команду UPDATE:
```bash
UPDATE employees SET department = 'Sales' WHERE first_name = 'Tobi' AND last_name = 'Barnett';
```
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_40_00" src="https://github.com/user-attachments/assets/2fe5dda8-e7ca-4257-8f6c-aacfc1f3206d" />

### 3. Зміна структури таблиці (Data Definition Language - DDL)
Для додавання нової колонки phone типу varchar(20) до таблиці employees, я застосувала команду ALTER TABLE:
```bash
ALTER TABLE employees ADD phone varchar(20);
```
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_47_10" src="https://github.com/user-attachments/assets/1343ca27-c063-4631-a116-83052429d834" />

### 4. Керування доступом (Data Control Language - DCL)
Останнім базовим завданням було надання всіх прав на таблицю grant_rights користувачу unauthorized_user:
```bash
GRANT ALL ON grant_rights TO unauthorized_user;
```
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_48_05" src="https://github.com/user-attachments/assets/5505c203-81dc-4f6b-8aa2-b117809b83c1" />

## Частина 2: Експлуатація SQL-ін'єкцій
5. Рядкова SQL-ін'єкція (String SQL Injection)
Завдання вимагало витягнути дані всіх користувачів, не знаючи конкретного імені. У поле Last Name я ввела пейлоад, який завжди повертає TRUE:
Smith' or '1'='1
Це змінило логіку запиту, і база даних повернула повний список акаунтів.
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_48_39" src="https://github.com/user-attachments/assets/876462c3-f4d5-4151-9a83-b4d51f1d93c9" />


### 7. Числова SQL-ін'єкція (Numeric SQL Injection)
Аналогічне завдання, але цього разу вразливим було числове поле User_Id. Я ввела значення:
1 OR '1'='1'
Запит знову успішно обробився, проігнорувавши оригінальні фільтри.
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_49_07" src="https://github.com/user-attachments/assets/16fa4e82-0182-4e12-abdb-aacf85b16e26" />


### 9. Порушення конфіденційності (Compromising Confidentiality)
Маючи доступ як John Smith (TAN: 3SL99A), я повинна була отримати зарплати всіх колег. У поле Employee Name я ввела:
Smith' OR '1'='1' -- 
Символи -- закоментували перевірку auth_tan у кінці запиту, що дозволило обійти механізм автентифікації транзакції.
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_49_52" src="https://github.com/user-attachments/assets/d3499e31-e5b5-4882-a716-e767d3e8455f" />

### 10. Порушення цілісності даних (Compromising Integrity - Query Chaining)
Наступним кроком була компрометація цілісності шляхом зміни власної зарплати. Використовуючи техніку "Query chaining" (об'єднання запитів через ;), я виконала одразу дві команди — оригінальний SELECT та мій зловмисний UPDATE:
'; UPDATE employees SET salary = 999999 WHERE last_name = 'Smith' --
В результаті зарплата мого профілю була успішно змінена.
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_52_59" src="https://github.com/user-attachments/assets/f7727b91-0e9e-4954-b2f0-7940e6136615" />

### 11. Порушення доступності (Compromising Availability)
Останнє завдання демонструє руйнівний потенціал SQLi. Аби знищити сліди своїх дій, я видалила таблицю з логами access_log. Пейлоад для поля пошуку:
'; DROP TABLE access_log --
Таблицю було видалено, що є прямим порушенням доступності системи.
<img width="1366" height="768" alt="Screenshot_2026-07-19_22_54_24" src="https://github.com/user-attachments/assets/3043db00-0fe7-4768-9501-1c8a2c73f0eb" />

## Висновки
Під час виконання цієї лабораторної роботи я успішно пройшла всі етапи модуля SQL Injection (intro). Я переконалася, що відсутність фільтрації вводу та використання непараметризованих запитів дозволяє зловмиснику маніпулювати логікою SQL. За допомогою простих пейлоадів (наприклад, ' OR '1'='1) та коментарів (--) мені вдалося обійти авторизацію, прочитати конфіденційні дані (зарплати колег), змінити записи у базі даних (підвищити собі зарплату) та навіть видалити цілі таблиці (знищити логи).

Це наочно демонструє критичність вразливостей класу A03:2021-Injection та необхідність використання Prepared Statements на етапі розробки.
