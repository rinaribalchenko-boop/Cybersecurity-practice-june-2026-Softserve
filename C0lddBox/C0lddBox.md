# Звіт з практичної роботи на тему: Підвищення привілеїв (Privilege Escalation) на машині ColddBox-Easy

## Мета роботи
Закріплення практичних навичок тестування на проникнення. Головне завдання — здійснити
підвищення привілеїв у системі Linux від стандартного користувача до суперкористувача (root),
використовуючи відомі вразливості конфігурації прав доступу та утиліти з набору GTFOBins.

## Практичне виконання

Після інсталювання образу у Virtual Box, запускаю віртуальну машину і відкриваю термінал. Прописую команду `sudo netdiscover -r 192.168.56/24`, щоб просканувати мережу, для знаходження потрібного ip

<img width="1366" height="768" alt="Screenshot_2026-07-26_16_02_12" src="https://github.com/user-attachments/assets/0a655394-9b54-48b6-bfbf-abfe60b92b59" />

Знайшлися дві ip-адреси, тепер застосую команду `sudo nmap -A --reason 192.168.56.103`, для сканування портів. Сканую другий, адже перший ip це службовий DHCP-сервер VirtualBox, і бачу що 80 порт відкритий

<img width="812" height="600" alt="image" src="https://github.com/user-attachments/assets/387b38dd-69db-424e-b018-1635dd917d06" />

У браузері відкриваю http://10.0.2.12, отримую:

<img width="1366" height="768" alt="Screenshot_2026-07-26_16_03_46" src="https://github.com/user-attachments/assets/2a4c80aa-b9df-4ded-9e06-5489df81800c" />

Проведжу дослідження структури вебсайту за допомогою gobuster:

`gobuster dir -u http://10.0.2.12/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

<img width="1366" height="726" alt="image" src="https://github.com/user-attachments/assets/f95d24ae-b906-4759-a44d-9ad68bf1498c" />

Перевіряю посилання і останнє дає нам такий результат:

<img width="1366" height="768" alt="Screenshot_2026-07-26_16_09_23" src="https://github.com/user-attachments/assets/1e5f7288-1c0e-4e1e-a267-5805de071df2" />

Розуміємо що є три користувачі C0ldd, Hugo та Philip, далі потрібно більш детально дослідити вордпресс на наявність додаткової інформації.

Командою `wpscan --url http://192.168.56.103/ -e` сканую цю адресу і шукаю інформацію про цих користувачів: 

<img width="867" height="421" alt="image" src="https://github.com/user-attachments/assets/e14d42e5-53a2-48a4-8f23-a0a4ecacd780" />

Тепер почнемо брут-форс за словником. Обираю користувача c0ldd тому що на ip-адресі яку ми сканували змінилось повідомлення на: «C0ldd, you changed Hugo's password, when you can send it to him so he can continue uploading his a...»
тобто обираємо користувача c0ldd для проведення брутфорсу тому, що цей запис вказує на його адміністративні привілеї. Той факт, що він може змінювати паролі інших користувачів (зокрема, користувача hugo), свідчить про те, що зламавши саме його акаунт,
ми отримаємо найбільші права та контроль над системою.

<img width="1366" height="733" alt="image" src="https://github.com/user-attachments/assets/c4dfb6df-16a7-46c5-9dd5-f1952a644856" />

Далі роблю логін в вордпрес та шукаю можливості впровадження коду для створення реверс-шелу. Для цього переходимо в Apearence->Editor.

<img width="1366" height="768" alt="Screenshot_2026-07-26_16_20_40" src="https://github.com/user-attachments/assets/1af32b13-31cf-4aac-bdaa-c8169b359748" />

Далі потрібно з'ясувати, який темплейт підходить для модифікації з метою встановлення реверс-шеллу. Візьмемо за основу публічний репозиторій: https://github.com/pentestmonkey/php-reverse-shell

<img width="1366" height="768" alt="Screenshot_2026-07-26_16_21_43" src="https://github.com/user-attachments/assets/3e0df7d2-2605-4aa8-9f1c-6cf3e4d94e09" />

Код реверс-шеллу найчастіше додають до шаблону "Footer" або "404" — почнемо з футера.
Змінюємо footer.php, додаючи туди код із php-reverse-shell.php та кастомізуючи його.

<img width="1366" height="768" alt="Screenshot_2026-07-26_16_31_24" src="https://github.com/user-attachments/assets/d448fe41-675e-4f4e-b805-6a931447dab4" />

Запускаємо слухач у терміналі Kali:
nc -lvnp 3421 (де параметри означають: -l — слухати, -v — деталі, -n — без DNS, -p — порт).

<img width="934" height="181" alt="image" src="https://github.com/user-attachments/assets/dfa43a5f-33a1-44b8-90db-a3bdb6d675de" />

Оновлюємо головну сторінку сервера та переходимо до покращення отриманої оболонки за допомогою pty.spawn("/bin/bash"), що робить її повноцінною інтерактивною сесією Bash.

<img width="924" height="393" alt="image" src="https://github.com/user-attachments/assets/a4c9c712-786e-407f-8ec5-ad48bc69a01a" />

# Підвищення привілеїв

Перебуваючи всередині системи, шукаємо важливі дані. Для WordPress базовим конфігураційним файлом є wp-config.php, який розташований у каталозі /var/www/html.

<img width="646" height="275" alt="image" src="https://github.com/user-attachments/assets/656c4f2a-39cb-4453-b6a1-7450dfa53031" />

Переглядаємо вміст файлу `wp-config.php` за допомогою утиліти `cat`:

<img width="760" height="768" alt="image" src="https://github.com/user-attachments/assets/aaaadead-806d-4a09-88c7-77a4ec6e41aa" />

Дізнаємося пароль: `cybersecurity`. Намагаємося увійти в систему від імені користувача c0ldd, використовуючи знайдені облікові дані:
`su c0ldd`:

<img width="462" height="196" alt="image" src="https://github.com/user-attachments/assets/61dc08cc-7dc3-40c1-b3ab-424db95c1ea6" />

Отримуємо результат. Переходимо до домашньої директорії користувача, аналізуємо її вміст та виводимо на екран знайдений файл.

<img width="586" height="177" alt="image" src="https://github.com/user-attachments/assets/617926f6-cef9-493f-b3c5-9790c1536d8b" />

Декодуємо отримані дані за допомогою сервісу www.base64decode.org та отримуємо привітання:

<img width="821" height="570" alt="image" src="https://github.com/user-attachments/assets/e068dbfd-8823-46ef-811f-05d2754873b3" />

Визначаємо наявні повноваження за допомогою команди:
`sudo -l`: 

<img width="762" height="228" alt="image" src="https://github.com/user-attachments/assets/640cf525-02ba-4056-96b5-836827f45a55" />

Ми можемо підвищити привілеї до рівня root, використовуючи будь-яку з трьох доступних команд. Протестуємо кожен із цих способів. Як допоміжний ресурс для обходу механізмів безпеки використовуватимемо базу знань GTFOBins:
`https://gtfobins.github.io/`

# Використання chmod
Небезпека полягає в тому, що якщо бінарному файлу дозволено виконуватися від імені суперкористувача через sudo, він не скидає підвищені привілеї. Це дозволяє використовувати його для доступу до файлової системи, підвищення прав або збереження привілейованого доступу.

Застосуємо цей метод, щоб змінити права доступу до захищеного файлу, який зазвичай недоступний для користувача з низькими привілеями. У нашому випадку цільовим є файл суперкористувача. Визначаємо змінну:
LFILE=root

Виконуємо команду:
sudo chmod 6777 $LFILE

Пояснення:

sudo — запуск команди від імені root.

chmod 6777: перша цифра 6 задає права власника (читання/запис), а 777 надає всім користувачам повні права на читання, запис і виконання.

Після виконання цієї послідовності команд отримуємо результат:

<img width="657" height="261" alt="image" src="https://github.com/user-attachments/assets/32266498-f4e5-49bd-8e60-08a49f0ade28" />

Виконую декодування base64 і також отримуємо привітання:

<img width="806" height="570" alt="image" src="https://github.com/user-attachments/assets/78dd8e3f-6724-4236-8aa0-9f7e385d642a" />

# Використання vim
Оскільки повноцінного root-доступу досі немає, переходимо до наступного кроку.

Запускаємо редактор vim через sudo:
`sudo vim`

<img width="565" height="297" alt="image" src="https://github.com/user-attachments/assets/00e1b9c1-55f6-4322-ac4e-d75438799176" />

<img width="340" height="113" alt="image" src="https://github.com/user-attachments/assets/2c79f950-aa37-4c14-b48d-fcfdd94a52f4" />

Отримуємо права root всередині `vim`. Для виходу використовуємо комбінацію Shift + ZZ, після чого натискаємо Enter (для виходу без збереження змін використовується Shift + zq).

4.3. Використання ftp
Звертаємося до рекомендацій із сайту GTFOBins (https://gtfobins.github.io/gtfobins/ftp/#sudo) та виконуємо команду для отримання оболонки:
`sudo ftp` -> `!/bin/bash`, і тепер також маємо root-доступ!

<img width="412" height="148" alt="image" src="https://github.com/user-attachments/assets/e00d6f5b-5253-40a3-8a8f-d5af2264f74c" />

# Висновок

У ході виконання лабораторної роботи було успішно проведено розвідку, виявлено обліковий запис адміністратора c0ldd та виконано підвищення привілеїв у системі до рівня `root`. Завдяки вразливостям у конфігурації бінарних файлів та використанню методів обходу обмежень (GTFOBins) через `chmod`, `vim` та `ftp`, було отримано повний контроль над цільовою системою. Це демонструє критичну важливість належного налаштування прав доступу та регулярного аудиту безпеки вебсерверів і додатків.

