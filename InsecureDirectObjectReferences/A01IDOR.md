# Звіт до лабораторної роботи: Insecure Direct Object References (IDOR) у WebGoat

## Середовище: Kali Linux, Docker engine, OWASP WebGoat container. 
## Мета роботи:
Дослідити вразливість типу **Insecure Direct Object References (IDOR)** на практиці за допомогою веб-додатка WebGoat та прокси-інструмента Burp Suite. Навчитися знаходити прямі посилання на об'єкти, аналізувати RESTful API запити та отримувати несанкціонований доступ до даних інших користувачів через маніпуляцію параметрами ідентифікаторів (userId).

## Хід виконання роботи

Як і у минулій лабораторній я використала локалльний Docker з WebGoat. Після запуску команди в терміналі я відкрила браузер і перейшла за адресою `http://127.0.0.1:8080/WebGoat`, де увійшла під своєю обліковою запискою. У бічному меню розділу **A1: Broken Access Control** я вибрала підрозділ **Insecure Direct Object References**.

<img width="1366" height="768" alt="Screenshot_2026-07-19_18_35_00" src="https://github.com/user-attachments/assets/e6424277-1554-4489-bf37-fa3d95d3747a" />

Першим кроком є здійснення легітимної автентифікації, після якої я починаю досліджувати можливі вектори обходу або експлуатації механізмів авторизації. Логін та пароль для цього облікового запису — tom та cat

Після проходження автентифікації переходжу до наступного завдання.

<img width="1366" height="768" alt="Screenshot_2026-07-19_18_36_32" src="https://github.com/user-attachments/assets/98914a45-2d61-4165-98c9-ada423d6e078" />

Далі відкриваю Burpsuite переходжу до браузеру, реєструюсь, і надсилаю запит, натиснувши "View profile". Далі аналузію у burpsuite, переглядаю історію httphistory знаходжу рядок IDOR/profile і шукаю відмінності:

<img width="1366" height="726" alt="image" src="https://github.com/user-attachments/assets/0f7faee8-06be-4c39-9da8-6b1a5de8912d" />

Тепер записую це у рядок для третього завдання:

<img width="1366" height="768" alt="Screenshot_2026-07-19_18_45_11" src="https://github.com/user-attachments/assets/00481ace-8936-4ff3-bc89-67a532bab90b" />

Далі я проаналізувала, що додаток використовує REST-шаблон для доступу до даних. Я самостійно сформувала альтернативний шлях, що починається з WebGoat, потім йде розділ IDOR, далі шлях до профілю, і наприкінці я ввела свій числовий ідентифікатор, натиснувши кнопку "Submit" для перевірки.

<img width="1366" height="735" alt="image" src="https://github.com/user-attachments/assets/37ad1847-8dc0-498c-a455-cf9b909a65b4" />

Тепер будемо переглядати інформацію профілю іншого користувача. Переходжу у BurpSuite логінюсь у браузері, надсилаю запит через кнопку "View profile". Шукаю відповідний запит у httphistory:

<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/996c84c7-c9b6-4003-9b7c-0e1d53312279" />

Надсилаю цей запит в **Intruder** і модифікую його, тобто замість зашифрованого ID профілю вписую свій, ставлю **Payloads -> numbers**; **Number range** -> **from 2342384 to 2342399**:

<img width="1366" height="728" alt="image" src="https://github.com/user-attachments/assets/1477fb40-0728-4ee3-8f9c-9f4a9eed75c3" />

Натискаю **Start attack** та аналізую рузультати:

<img width="1366" height="724" alt="image" src="https://github.com/user-attachments/assets/1b7c66bb-0e63-418c-b497-857aed952dce" />

Щоб відредагувати профіль, я відправляю позитивний запит (з минулого скріна) до **Reapeter** роблю з **GET -> PUT**; Змінюю **Connection-Type -> application/json** (для того щоб сервер розпізнав формат надісланих даних та обробив їх коректно); І потім прописую що саме хочу змінити, перезаписую інформацію, роль та колір. Також потрібно мати актуальний (час обмежений) **JSESSIONID**. Натискаю **Send** і  дивлюсь що вийшло: 

<img width="1366" height="726" alt="image" src="https://github.com/user-attachments/assets/bbffe3ad-7bde-4888-8238-f073420ed9cf" />

І як результат завдання зараховано!

<img width="1366" height="735" alt="image" src="https://github.com/user-attachments/assets/122da467-f9c1-4a5b-b017-f49976dc924c" />

