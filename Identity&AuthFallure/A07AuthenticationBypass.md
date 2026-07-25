# Звіт з виконання завдання: A07 Authentication Bypass (WebGoat)

## 1. Мета завдання
Дослідити вразливості логіки аутентифікації у вебзастосунках на прикладі обходу перевірки секретних питань (2FA Password Reset / Security Questions). Навчитися використовувати **Burp Suite** для перехоплення та модифікації HTTP-запитів з метою експлуатації логічних помилок на стороні сервера.

## 2. Теоретичні відомості
**Authentication Bypass (Обхід аутентифікації)** — це клас вразливостей, який дозволяє зловмиснику отримати доступ до системи або підвищити привілеї, оминаючи передбачені механізми перевірки особи. У контексті OWASP Top 10 (A07:2021-Identification and Authentication Failures), такі проблеми часто виникають через:
*   Неправильну реалізацію логіки перевірки на бекенді (наприклад, алгоритм пропускає користувача, якщо параметр валідації взагалі відсутній).
*   Довіру до даних, що надходять від клієнта (Client-side trust).
*   Відсутність підходу **"Default Deny"** (заборонити за замовчуванням) під час написання умов перевірки.

---

## 3. Хід виконання (Практична частина)
Для початку роботи я відкрила термінал та запустила контейнер із сервером WebGoat за допомогою Docker. У команді було налаштовано прокидання портів (8080 та 9090) на локальний інтерфейс `127.0.0.1`, а також вказано часовий пояс для коректної роботи токенів у завданнях:

```bash
sudo docker run -p 127.0.0.1:8080:8080 -p 127.0.0.1:9090:9090 -e TZ=Europe/Amsterdam webgoat/webgoat
```
<img width="1366" height="768" alt="Screenshot_2026-07-18_22_31_52" src="https://github.com/user-attachments/assets/44fa2d9a-323b-4b92-9c3d-b42e85138392" />

Після виконання цієї команди сервер WebGoat стає доступним за адресою, на офіційному сайті WebGoat. Зазвичай це http://127.0.0.1:8080/WebGoat.

Я перейшла за посиланням `http://127.0.0.1:8080/WebGoat` у браузері. Після авторизації в системі я відкрила бічне меню та перейшла до потрібного розділу, де знаходиться завдання **Authentication Bypass**
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/0eaf493b-c370-4632-a7df-fa1672d95bf5" />
### Аналіз функціоналу
У завданні "Authentication Bypasses" нам пропонується форма верифікації облікового запису. Замість пароля система вимагає відповісти на два секретні питання (`secQuestion0` та `secQuestion1`), які були задані під час реєстрації.
Тепер запускаю BurpSuite, логінюсь через браузер, заповнюю поля секретних питань, але перед відправкою у burpsuite натискаю Intercept -> on. Натискаю Submit у браузері.
<img width="1366" height="768" alt="Screenshot_2026-07-25_18_26_24" src="https://github.com/user-attachments/assets/ec05776a-c863-4b79-81bd-18b8da9c7c26" />
### Перехоплення та дослідження запиту
У вкладці `Proxy -> Intercept` ми бачимо сформований POST-запит, який браузер намагається відправити на сервер. Тіло запиту (Body) виглядає приблизно так:
<img width="1366" height="768" alt="Screenshot_2026-07-25_18_29_06" src="https://github.com/user-attachments/assets/ac9f3136-9a3d-47e9-a9ad-0602f1a053f9" />
Якщо ми просто пропустимо цей запит (Forward), сервер поверне помилку, оскільки наші випадкові відповіді не збігаються з хешами в базі даних. Тому я надсилаю цей запит у Repeater:
<img width="1366" height="768" alt="Screenshot_2026-07-25_18_39_36" src="https://github.com/user-attachments/assets/27bf525c-b083-4ad9-ac7d-5a8f3e965b8b" />
Якщо ми натиснемо Send, то у Response побачимо відповідь "Не зовсім так, спробуйте ще раз":
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/62efd2e9-e872-4281-aeaf-14ea7b23c7ae" />
Якщо ми видалимо строку з секретними питаннями, то також отмимуємо помилку:
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/58815663-ef60-48d5-92d5-b67fa17450c5" />
А якщо змінимо `secQuestion0` та `secQuestion1` на `secQuestionA`, `secQuestionB` і натиснемо Send, то тоді ми за логінемося у аккаунт:
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/6a712ea7-6b5a-4655-a22e-80b2887de7c1" />
Переходимо знову до сторінки Intercept та натискаємо Forward:
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/5df46cd9-a9e1-4a58-ad04-3792b51bdf2b" />
І тепер у браузері бачимо, що наше завдання зараховане, і ми можемо змінити пароль на аккаунті:
<img width="1366" height="768" alt="Screenshot_2026-07-25_18_42_08" src="https://github.com/user-attachments/assets/027673fc-af20-46c7-b08d-fc63e76663b9" />




