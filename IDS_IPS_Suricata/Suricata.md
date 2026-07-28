# Звіт до практичної роботи за темою: Встановлення та налаштування системи виявлення вторгнень Suricata


## Мета роботи:
Ознайомитися з системою виявлення вторгнень (IDS) Suricata, встановити її, оновити правила, налаштувати конфігураційний файл та перевірити працездатність служби.

---
### Хід роботи
#### Встановлення Suricata

Перед початком роботи встановила віртуальну машину на Linux Ubuntu 24.04 з відповідними налаштуваннями

<img width="1366" height="768" alt="Screenshot_2026-07-26_23_06_41" src="https://github.com/user-attachments/assets/3b410af6-00f4-4d9a-a5a1-22de6ee815af" />

Далі було встановлено систему Suricata за допомогою менеджера пакетів Ubuntu. 
```bash
sudo apt update
sudo apt install suricata -y
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/95283c50-ae34-404f-b38d-ecef27472af1" />

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/11753f51-2c9c-4050-a36d-38aa58b1a63f" />

Команди встановлює IDS Suricata разом із необхідними бібліотеками та службами.

#### Перевірка встановлення

Після завершення інсталяції було перевірено версію програми:

```
suricata --build-info
suricata '--version`'
```
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/e4042f78-b4de-4cd6-9854-e44fdb4fbf39" />

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/6bb7eb9f-104f-47b2-8d2f-314d7c598dae" />

Перевіряю ще раз службу:

`sudo system status suricata`

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/f2e1b5b9-47e0-4334-9580-9eb33c33b39a" />

Прописала також `sudo suricata-update` - використовується для автоматичного завантаження та оновлення бази правил (сигнатур) Suricata. Вона отримує актуальні правила з офіційних джерел, формує файл suricata.rules та зберігає його у системі. 
Це дозволяє Suricata виявляти сучасні мережеві атаки, шкідливий трафік та інші загрози. і `ls -lh /var/lib/suricata/rules` - використовується для перегляду вмісту каталогу з правилами Suricata та перевірки успішного створення файлів правил після їх оновлення. 
Вона дозволяє переконатися, що база сигнатур була успішно завантажена та збережена у системі.

<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/ecb40fd9-38c6-486f-8a87-b7f83c51509e" />




