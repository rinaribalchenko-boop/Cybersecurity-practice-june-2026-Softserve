# Звіт з практичної роботи: Дослідження веб-вразливостей в OWASP Juice Shop

**Виконала:** Рибальченко Ріна  
**Мета роботи:** Ознайомитися з основними веб-вразливостями (згідно з OWASP Top 10) та відпрацювати навички їх експлуатації на базі навчального майданчика OWASP Juice Shop.

---
## Виконання завдань
Відкриваю термінал та прописую команду 
```bash
sudo docker run --rm -p 3000:3000 bkimminich/juice-shop 
```
тобто запустила контейнер із сервером JuiceShop за допомогою Docker. 

<img width="1366" height="768" alt="Screenshot_2026-07-19_23_17_37" src="https://github.com/user-attachments/assets/b311b7e3-5806-44f2-b6cc-72863ca5ecb7" />

Відкриваю браузер та вводжу адресу: http://localhost:3000

<img width="1366" height="768" alt="Screenshot_2026-07-19_23_18_29" src="https://github.com/user-attachments/assets/0d279890-e40c-4539-85a7-25a4766d7fa4" />

## 1. Завдання: Score Board (Дошка результатів)
**Категорія:** Miscellaneous (Різне) | **Складність:** ⭐

**Опис завдання:** 
За замовчуванням сторінка з дошкою результатів (де відображається прогрес виконання всіх тасок) прихована від користувача. Мета — знайти її URL-адресу та відкрити.

**Хід виконання:**
1. Оскільки додаток є Single Page Application (SPA) і написаний на Angular, логіка маршрутизації знаходиться на стороні клієнта.
2. Я відкрила Інструменти розробника (F12) у браузері і перейшла на вкладку **Sources** (Джерела).
3. У файлі `main.js` (або `main-es2015.js`) за допомогою пошуку (Ctrl+F) знайдено ключове слово `scoreBoard`.
4. У коді виявлено шлях маршрутизатора: `'score-board'`
   <img width="1366" height="768" alt="Screenshot_2026-07-19_23_25_57" src="https://github.com/user-attachments/assets/09dab017-d1e1-4510-bf12-7969a107fc37" />
6. Додано знайдений шлях до базового URL: `http://localhost:3000/#/score-board` і здійснено перехід:
<img width="1366" height="768" alt="Screenshot_2026-07-19_23_26_27" src="https://github.com/user-attachments/assets/1379a7ec-1512-40aa-846e-d87a97c77b38" />

---
## 2. Завдання: DOM XSS
**Категорія:** XSS (Міжсайтовий скриптинг) | **Складність:** ⭐

**Опис завдання:** 
Виконати атаку DOM-based Cross-Site Scripting, змусивши браузер виконати сторонній JavaScript-код.

**Хід виконання:**
1. На головній сторінці використано рядок пошуку (іконка лупи).
2. Замість звичайного тексту введено payload: `<iframe src="javascript:alert('XSS_Success')">`
   *(Примітка: звичайний тег `<script>` тут не спрацьовує через базову фільтрацію фреймворку, але iframe ігнорується захистом).*
3. Натиснуто Enter.

Введений пошуковий запит передається в URL-параметр `?q=...` і безпосередньо (без належного санітування чи екранування символів `<` і `>`) вбудовується в Document Object Model (DOM) сторінки. Браузер зчитує цей ввід не як текст, а як виконуваний HTML-код.

На екрані з'явилося спливаюче вікно (alert box) із текстом `XSS_Success`.

<img width="1366" height="768" alt="Screenshot_2026-07-19_23_36_20" src="https://github.com/user-attachments/assets/e7471782-ca4e-459d-9165-023139bc1bd8" />

Роблю відправку пейлоада, втавляю цей код у пошукову стрічку та натискаю *Enter*:
```bash
<iframe width="100%" height="166" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/771984076&color=%23ff5500&auto_play=true&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true"></iframe>
```
<img width="1366" height="768" alt="Screenshot_2026-07-19_23_40_22" src="https://github.com/user-attachments/assets/606dd4eb-35c6-4aab-9170-b61428ef378f" />

---

## 3. Завдання: Confidential Document 
**Категорія:** Sensitive Data Exposure | **Складність:** ⭐

**Опис завдання:** 
Отримати доступ до внутрішнього конфіденційного документа компанії, який не має бути доступним звичайним користувачам.

**Хід виконання:**
1. Шукаю вікриті директорії. Прописую команду gobuster: `gobuster dir -u http://127.0.0.1:3000/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
   <img width="1366" height="768" alt="Screenshot_2026-07-19_23_43_25" src="https://github.com/user-attachments/assets/2d8676f7-32ef-44d6-bc06-119aa28e3223" />
2. Вношу зміни, які нам рекомендуються і знову запускаю сканування:
   <img width="1366" height="768" alt="Screenshot_2026-07-19_23_47_13" src="https://github.com/user-attachments/assets/9caf203b-164e-4c19-8894-0695970bdc09" />
3. Випадково проходжу Error Handing, зламавши джусі шоп:
   <img width="1366" height="768" alt="Screenshot_2026-07-20_00_08_44" src="https://github.com/user-attachments/assets/521ec001-2344-4e3a-9776-30afabfd648c" />
5. Перезавантажую контейнер juice shop та переходжу до знайденого ftp
<img width="1366" height="768" alt="Screenshot_2026-07-20_00_00_16" src="https://github.com/user-attachments/assets/e5dd3ca6-afa5-4f16-8e8f-f9c7e9b9ee25" />
7. Переходжу до директорії http://localhost:3000/ftp, тепер намагаюсь завантажити файл, додаючи до запиту %2500.md
   <img width="1366" height="768" alt="Screenshot_2026-07-20_00_16_24" src="https://github.com/user-attachments/assets/00956a35-9518-4fec-8520-aca2d5572ca7" />


**Аналіз вразливості:**
Проблема полягає у двох конфігураційних помилках:
* Сервер дозволяє перегляд вмісту директорій (Directory Listing).
* Відсутній контроль доступу до статичних файлів — сервер не перевіряє права користувача перед завантаженням документа.

**Результат:**
Як результат документ завантажився і таск у джусі шопі також пройдений:
<img width="1366" height="768" alt="Screenshot_2026-07-20_00_18_18" src="https://github.com/user-attachments/assets/33772b21-cdec-484a-87e7-d8e20311ca77" />
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/ba5381fd-bd4d-4a16-8cc9-7259f06b6f7e" />

---
## 4. Завдання: Login Admin (Вхід від імені адміністратора)
**Категорія:** Injection (SQL Ін'єкція) | **Складність:** ⭐⭐

**Опис завдання:** 
Отримати доступ до облікового запису адміністратора (`admin@juice-sh.op`), не знаючи його справжнього пароля, використовуючи вразливість SQL-ін'єкції.

**Хід виконання:**
1. Здійснено перехід на сторінку авторизації (`127.0.0.1:3000/#/login`).
2. Для експлуатації SQL-ін'єкції у поле `Email` було введено класичний payload: `' OR 1=1--`
3. У поле `Password` введено довільний набір символів (наприклад, `12345`).
4. Натиснуто кнопку "Log in".
<img width="1366" height="768" alt="Screenshot_2026-07-20_00_19_24" src="https://github.com/user-attachments/assets/3977d86c-6eb1-4ae8-bb69-8185ca46fc52" />


**Аналіз вразливості (Чому це працює):**
Бекенд формує SQL-запит до бази даних приблизно таким чином:  
`SELECT * FROM Users WHERE email = '<ввід_користувача>' AND password = '<пароль>'`

Після підстановки мого payload запит набув вигляду:  
`SELECT * FROM Users WHERE email = '' OR 1=1--' AND password = '...'`
* Одинарна цитата `'` "закриває" поле email.
* Конструкція `OR 1=1` завжди повертає `True`.
* Символи `--` коментують усю подальшу частину запиту (включно з перевіркою пароля).
Оскільки запис адміністратора зазвичай є першим у базі даних (id=1), база повертає його, і система нас авторизує.

**Результат:**
Успішна авторизація під акаунтом адміністратора.
<img width="1366" height="768" alt="Screenshot_2026-07-20_00_20_20" src="https://github.com/user-attachments/assets/c11ea94d-59ea-4698-bbce-1fc5bdd72cef" />
Переглядаю чи пройдено у juice shop: 
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/340a73b8-9e2a-4042-a57f-6ba29a2c9d9b" />

--- 

## 5. Завдання: Legacy Typosquatting
**Категорія:** Vulnerable Components (Вразливі компоненти) | **Складність:** ⭐⭐⭐⭐

**Опис завдання:** 
Повідомити розробників магазину про те, що в минулому вони стали жертвою атаки "Typosquatting" (використання пакета з навмисно зміненою назвою) при виборі сторонніх бібліотек. Для цього потрібно знайти назву шкідливого пакета і надіслати її через форму зворотного зв'язку.

**Хід виконання:**
1. Із попередніх етапів дослідження (завдання з доступом до `/ftp`) було виявлено, що на сервері увімкнено Directory Listing.
2. Перейшовши за адресою `http://localhost:3000/ftp`, я знайшла файл `package.json.bak`. Розширення `.bak` свідчить про те, що це стара (legacy) резервна копія конфігурації залежностей проєкту.
3. Завантаживши та відкривши цей файл, я проаналізувала блок `dependencies` на наявність підозрілих назв npm-пакетів.
   <img width="1366" height="768" alt="Screenshot_2026-07-25_15_17_02" src="https://github.com/user-attachments/assets/573ea31d-2c96-4ac1-973b-80b991366472" />
4. Відкрила сайт npmjs.com , щоб перевірити легітимність бібліотек
   <img width="1366" height="768" alt="Screenshot_2026-07-25_15_34_57" src="https://github.com/user-attachments/assets/acdceded-c46d-429e-a856-7d087bf7ed15" />
6. Серед легітимних бібліотек було виявлено пакет `epilogue-js`. Оригінальна популярна бібліотека має назву `epilogue` (без `-js`). Це класична ознака підробленого пакета.
   <img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/75cb5309-30da-4a20-91bf-f9365ddd8697" />
   <img width="1366" height="768" alt="Screenshot_2026-07-25_15_35_53" src="https://github.com/user-attachments/assets/be8b9d78-e1ac-4fff-bbf8-a81e59988ebe" />
8. Для виконання завдання я перейшла на сторінку "Customer Feedback" (`/#/contact`).
9. У полі коментаря (Comment) я написала назву виявленого пакета: `epilogue-js`. Заповнила інші поля (рейтинг) та надіслала форму.
    <img width="1366" height="768" alt="Screenshot_2026-07-25_15_38_56" src="https://github.com/user-attachments/assets/62548e9c-fc3b-40c6-a33b-a757faefb202" />

**Аналіз вразливості (Чому це працює і в чому небезпека):**
Typosquatting — це різновид атаки на ланцюг постачання. Зловмисники публікують у відкритих репозиторіях (наприклад, npm) шкідливий код під іменами, що дуже схожі на популярні бібліотеки (різниця в одну букву, переставлені букви, або доданий суфікс на кшталт `-js`). 

Розробник при налаштуванні середовища може випадково зробити помилку під час `npm install` і завантажити бекдор. Вразливість (згідно з OWASP — Vulnerable and Outdated Components) полягає у відсутності автоматизованого аудиту сторонніх бібліотек на етапі CI/CD та залишенні на сервері старих бекапів (`.bak`), що дозволяє зловмиснику зібрати інформацію про стек технологій та історичні вразливості системи (Information Disclosure).

**Результат:**
Після відправки відгуку з текстом "epilogue-js" система зарахувала завдання, і з'явилося сповіщення про успішне проходження таски на 4 зірки.
<img width="1366" height="768" alt="Screenshot_2026-07-25_15_39_02" src="https://github.com/user-attachments/assets/d96d746c-fb95-4f9c-a3f8-2643dde30db1" />
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/438f0b9a-f15f-44c5-a80b-481a207b93b9" />

## Висновки
Під час виконання лабораторної роботи на базі OWASP Juice Shop було на практиці перевірено механізми виникнення поширених веб-вразливостей. Навчившись аналізувати клієнтський код, працювати з URL-параметрами та маніпулювати SQL-запитами, було успішно експлуатовано базові вразливості, що демонструє важливість ретельної валідації вхідних даних та правильної конфігурації серверів.
