# Звіт до практичної роботи за темою: Встановлення та налаштування системи виявлення вторгнень Suricata


## Мета роботи:
Ознайомитися з системою виявлення вторгнень (IDS) Suricata, встановити її, оновити правила, налаштувати конфігураційний файл та перевірити працездатність служби.

---
### Хід роботи
#### Встановлення Suricata

Перед початком роботи було створено та налаштовано віртуальну машину з операційною системою Ubuntu 24.04, яка рекомендована викладачем для виконання лабораторної роботи.

<img width="1366" height="768" alt="Screenshot_2026-07-26_23_06_41" src="https://github.com/user-attachments/assets/3b410af6-00f4-4d9a-a5a1-22de6ee815af" />

Для встановлення системи виявлення вторгнень Suricata було виконано такі команди:
```bash
sudo apt update
sudo apt install suricata -y
```
Команда `sudo apt update` оновлює список доступних пакетів операційної системи, що дозволяє отримати найновіші версії програм.
Команда `sudo apt install suricata -y` встановлює систему виявлення вторгнень Suricata разом із необхідними бібліотеками та службами.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/95283c50-ae34-404f-b38d-ecef27472af1" />

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/11753f51-2c9c-4050-a36d-38aa58b1a63f" />

#### Перевірка встановлення

Після завершення інсталяції було перевірено встановлену версію Suricata.
```
suricata --build-info
suricata --version
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/e4042f78-b4de-4cd6-9854-e44fdb4fbf39" />

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/6bb7eb9f-104f-47b2-8d2f-314d7c598dae" />

Команда `suricata --build-info` виводить інформацію про версію програми, підтримувані модулі та бібліотеки.
Команда `suricata --version` дозволяє швидко перевірити встановлену версію Suricata.

Після встановлення було перевірено стан служби Suricata:

`sudo systemctl status suricata`

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/f2e1b5b9-47e0-4334-9580-9eb33c33b39a" />

Команда відображає поточний стан служби, інформацію про її запуск та дозволяє переконатися, що Suricata працює коректно.


Для отримання актуальних сигнатур було виконано команду:

`sudo suricata-update`

<img width="901" height="467" alt="image" src="https://github.com/user-attachments/assets/07b9e4e1-77ef-4278-b708-5e9ea795f95b" />


Команда автоматично завантажує та оновлює базу правил Suricata з офіційних джерел. Отримані сигнатури використовуються для виявлення сучасних мережевих атак та інших загроз.

Після завершення оновлення було перевірено наявність файлів правил.

`ls -lh /var/lib/suricata/rules`

<img width="762" height="151" alt="image" src="https://github.com/user-attachments/assets/8080fb89-33a3-4ab0-9bea-8bc8dc8ea90e" />

Для визначення активного мережевого інтерфейсу та IP-адреси було виконано команди:

```
ip a
ip link
```

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/8f0ab6fb-937a-42ba-abf5-407a0075c514" />
Команда `ip a` відображає інформацію про мережеві інтерфейси та їхні IP-адреси.
Команда `ip link` показує список мережевих інтерфейсів і їхній поточний стан (активний чи неактивний).


Після визначення IP-адреси було відкрито конфігураційний файл Suricata:
`sudo nano /etc/suricata/suricata.yaml`

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/f12edb2c-7715-4649-997c-e1d4c9bf4315" />

У параметрі HOME_NET було встановлено IP-адресу локальної мережі. Це необхідно для того, щоб Suricata правильно розпізнавала трафік, який належить до внутрішньої мережі.

Після внесення змін конфігурацію було перевірено командою:
`sudo suricata -T -c /etc/suricata/suricata.yaml` 
Команда перевіряє правильність конфігураційного файлу без запуску системи.
Після успішної перевірки службу було перезапущено.

`sudo systemctl restart suricata`

Потім було повторно перевірено її стан.

`sudo systemctl status suricata`

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/4934e9a3-1160-4afe-84ee-94599685dcd8" />

### Створення власних правил Suricata

Після успішного встановлення та налаштування Suricata було створено власні правила виявлення мережевого трафіку.

Спочатку було відкрито файл локальних правил:

`sudo nano /etc/suricata/rules/local.rules`

Файл `local.rules` призначений для збереження власних сигнатур користувача. На відміну від стандартних правил, які автоматично оновлюються командою suricata-update, цей файл дозволяє створювати власні правила без ризику їх втрати під час оновлення бази сигнатур.

### Створення першого правила

До файлу було додано правило:

`alert icmp any any -> $HOME_NET any (msg:"Rina Custom ICMP Detected"; sid:1000001; rev:1;)`,

де:

`alert` — дія, яка наказує Suricata сформувати повідомлення про подію;

`icmp` — правило застосовується до протоколу ICMP;

`any any` — правило працює для будь-якої IP-адреси та будь-якого порту джерела;

`->` — напрямок руху трафіку;

`$HOME_NET any` — призначенням є внутрішня мережа, визначена параметром HOME_NET;

`msg` — повідомлення, яке буде записане до журналу;

`sid` — унікальний ідентифікатор правила;

`rev` — номер редакції правила.

Дане правило дозволяє виявляти ICMP-трафік, який виникає, наприклад, під час виконання команди `ping`. Такі пакети часто використовуються для перевірки доступності вузлів мережі.

### Створення другого правила

Було створено ще одне правило:

`alert http any any -> $HOME_NET any (msg:"Rina HTTP Traffic"; sid:1000002; rev:1;)`

Це правило формує повідомлення при виявленні HTTP-трафіку. Воно дозволяє контролювати звернення до вебресурсів.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/7cdf1ad2-9395-41e7-a9f9-18958457c745" />

Зміни було збережено комбінацією Cntrl+O; Enter, а після вийшли з редактора Cntrl+X

### Підключення локальних правил

Після створення правил необхідно повідомити Suricata, що потрібно використовувати файл `local.rules`

Для цього було відкрито конфігураційний файл:

`sudo nano /etc/suricata/suricata.yaml`

У розділі

`rule-files`:

було додано рядок:

`- local.rules`

Після цього Suricata почала завантажувати власні правила разом із основною базою сигнатур.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/faa50e5c-f06b-4ebf-bbe5-72cec7d821e8" />

### Перевірка конфігурації

Після внесення змін було виконано перевірку конфігураційного файлу.

`sudo suricata -T -c /etc/suricata/suricata.yaml`

Команда перевіряє правильність конфігурації та всіх підключених правил без запуску Suricata.

У результаті було отримано повідомлення:

`Configuration provided was successfully loaded.`

Це підтвердило відсутність помилок у конфігурації.

<img width="861" height="128" alt="image" src="https://github.com/user-attachments/assets/b2205e69-b2ce-459f-8138-6f67a0dace2e" />

### Перезапуск служби

Щоб застосувати внесені зміни, службу було перезапущено.

`sudo systemctl restart suricata`

Після цього ще раз було перевірено її стан.

`sudo systemctl status suricata`

Статус active (running) підтвердив, що служба працює коректно.

<img width="1280" height="419" alt="image" src="https://github.com/user-attachments/assets/95235d89-9d39-4f4d-ab48-2d85b67db1e8" />

### Перевірка створених правил

Для перегляду повідомлень про спрацювання правил було відкрито журнал Suricata.

`sudo tail -f /var/log/suricata/fast.log`

Також використовувався журнал у форматі JSON.

`sudo tail -f /var/log/suricata/eve.json | jq 'select(.event_type=="alert")'`

Команда дозволяє в режимі реального часу відображати лише повідомлення типу alert.

### Перевірка ICMP-правила

Для створення ICMP-трафіку було виконано команду:

`ping 8.8.8.8`

У журналі Suricata з'явилося повідомлення

`Rina Custom ICMP Detected`

що підтвердило успішне спрацювання створеного правила.

<img width="744" height="361" alt="image" src="https://github.com/user-attachments/assets/b94c2287-fb7a-4fe4-a36f-6a9bb7f140ad" />

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/39b90429-7a67-423e-8345-1bd4e8d6e07b" />

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/2757b3ec-682f-44b7-bc73-6d8766b4acd0" />

### Перевірка HTTP-правила

Для перевірки другого правила було створено HTTP-трафік шляхом відкриття вебсторінки або виконання HTTP-запиту.

У журналі Suricata було зафіксовано повідомлення

`Rina HTTP Traffic`

що підтвердило правильність роботи другого правила.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/5d949cc6-403a-4c0e-b3ae-a7f6cdf9e87c" />

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/d031598a-2b14-4b7a-a4ad-9c432f149e2b" />

### Висновок

Під час виконання лабораторної роботи було успішно створено та налаштовано власні правила виявлення мережевого трафіку в системі Suricata. Було реалізовано два правила: для контролю ICMP-трафіку та HTTP-трафіку. Після підключення локального файлу правил, перевірки конфігурації та перезапуску служби було підтверджено їхню працездатність. Отримані результати показали, що Suricata успішно виявляє події відповідно до створених сигнатур, що дозволило закріпити практичні навички роботи із системою виявлення вторгнень та створення власних правил аналізу мережевого трафіку.
