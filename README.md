# Домашнее задание к занятию "`Система мониторинга Zabbix`" - `Коноплёв Александр`


### Инструкция по выполнению домашнего задания

   1. Сделайте `fork` данного репозитория к себе в Github и переименуйте его по названию или номеру занятия, например, https://github.com/имя-вашего-репозитория/git-hw или  https://github.com/имя-вашего-репозитория/7-1-ansible-hw).
   2. Выполните клонирование данного репозитория к себе на ПК с помощью команды `git clone`.
   3. Выполните домашнее задание и заполните у себя локально этот файл README.md:
      - впишите вверху название занятия и вашу фамилию и имя
      - в каждом задании добавьте решение в требуемом виде (текст/код/скриншоты/ссылка)
      - для корректного добавления скриншотов воспользуйтесь [инструкцией "Как вставить скриншот в шаблон с решением](https://github.com/netology-code/sys-pattern-homework/blob/main/screen-instruction.md)
      - при оформлении используйте возможности языка разметки md (коротко об этом можно посмотреть в [инструкции  по MarkDown](https://github.com/netology-code/sys-pattern-homework/blob/main/md-instruction.md))
   4. После завершения работы над домашним заданием сделайте коммит (`git commit -m "comment"`) и отправьте его на Github (`git push origin`);
   5. Для проверки домашнего задания преподавателем в личном кабинете прикрепите и отправьте ссылку на решение в виде md-файла в вашем Github.
   6. Любые вопросы по выполнению заданий спрашивайте в чате учебной группы и/или в разделе “Вопросы по заданию” в личном кабинете.
   
Желаем успехов в выполнении домашнего задания!
   
### Дополнительные материалы, которые могут быть полезны для выполнения задания

1. [Руководство по оформлению Markdown файлов](https://gist.github.com/Jekins/2bf2d0638163f1294637#Code)

zabbix-net (192.168.100.0/24)
├── zabbix-mysql (Database)
├── zabbix-server (Monitoring Server)
├── zabbix-web (Web Interface)
├── zabbix-agent-1 (Agent 1)
└── zabbix-agent-2 (Agent 2)

Задание 1
Установите Zabbix Server с веб-интерфейсом.

Процесс выполнения
Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
Установите PostgreSQL. Для установки достаточна та версия, что есть в системном репозитороии Debian 11.
Пользуясь конфигуратором команд с официального сайта, составьте набор команд для установки последней версии Zabbix с поддержкой PostgreSQL и Apache.
Выполните все необходимые команды для установки Zabbix Server и Zabbix Web Server.
Требования к результатам
Прикрепите в файл README.md скриншот авторизации в админке.
Приложите в файл README.md текст использованных команд в GitHub.

# 1. Устанавливаем PostgreSQL
sudo apt install -y postgresql postgresql-contrib
# 2. Запускаем и добавляем в автозагрузку
sudo systemctl start postgresql
sudo systemctl enable postgresql

# 3. Создаём базу данных и пользователя Zabbix
# Переключаемся на пользователя postgres
sudo -u postgres psql
# В командной строке PostgreSQL выполняем:
CREATE USER zabbix WITH PASSWORD 'zabbix' CREATEDB;
CREATE DATABASE zabbix OWNER zabbix;
GRANT ALL PRIVILEGES ON DATABASE zabbix TO zabbix;
\q
# 4. Устанавливаем репозиторий Zabbix
wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb
sudo apt update
# 5. Устанавливаем Zabbix  
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo dpkg -i zabbix-release_7.0-1+ubuntu24.04_all.deb
sudo apt update
sudo apt install -y zabbix-server-pgsql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent 
# 6. Устанавливаем PHP и необходимые модули
sudo apt install -y php8.1-pgsql php8.1-fpm
# 7. Импортируем схему базы данных
sudo zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix
# 8. Редактируем конфигурационный файл
sudo nano /etc/zabbix/zabbix_server.conf

# Изменяем следующие параметры:
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix

# 9. Перезапускаем сервисы:
sudo systemctl enable zabbix-server zabbix-agent nginx php8.1-fpm
# Проверяем статус
sudo systemctl status zabbix-server
<img width="858" height="366" alt="Перезапуск сервисов и проверка состояния" src="https://github.com/user-attachments/assets/1bcee2cf-3fba-4818-a36c-f6ad7896a579" />
# 10. В браузере открываем графический интерфейс и проходим регистрацию
http://192.168.1.103/zabbix/
<img width="1642" height="753" alt="Успешная регистрация в Zabbix" src="https://github.com/user-attachments/assets/c7e67312-6708-49c6-8f13-3595697f180f" />


Задание 2
Установите Zabbix Agent на два хоста.

Процесс выполнения
Выполняя ДЗ, сверяйтесь с процессом отражённым в записи лекции.
Установите Zabbix Agent на 2 вирт.машины, одной из них может быть ваш Zabbix Server.
Добавьте Zabbix Server в список разрешенных серверов ваших Zabbix Agentов.
Добавьте Zabbix Agentов в раздел Configuration > Hosts вашего Zabbix Servera.
Проверьте, что в разделе Latest Data начали появляться данные с добавленных агентов.
Требования к результатам
Приложите в файл README.md скриншот раздела Configuration > Hosts, где видно, что агенты подключены к серверу
Приложите в файл README.md скриншот лога zabbix agent, где видно, что он работает с сервером
Приложите в файл README.md скриншот раздела Monitoring > Latest data для обоих хостов, где видны поступающие от агентов данные.
Приложите в файл README.md текст использованных команд в GitHub
Поле для вставки кода...


# 1. Запускаем агентов

sudo docker run -d \
  --name zabbix-agent-1 \
  -e ZBX_HOSTNAME="zabbix-agent-1" \
  -e ZBX_SERVER_HOST="192.168.1.103" \
  -e ZBX_SERVER_PORT="10051" \
  -p 10052:10050 \
  zabbix/zabbix-agent2:alpine-6.4-latest

sudo docker run -d \
  --name zabbix-agent-2 \
  -e ZBX_HOSTNAME="zabbix-agent-2" \
  -e ZBX_SERVER_HOST="192.168.1.103" \
  -e ZBX_SERVER_PORT="10051" \
  -p 10053:10050 \
  zabbix/zabbix-agent2:alpine-6.4-latest

Подключённые агенты в Zabbix Скриншот 1
<img width="1603" height="639" alt="Агенты подключённые к серверу" src="https://github.com/user-attachments/assets/3ce8cf21-cf73-41a0-8af9-52e94a8e4618" />
Проверка подключения агентов к серверу Скриншот 2
<img width="862" height="378" alt="Проверка подключения агентов к серверу" src="https://github.com/user-attachments/assets/475d60a0-3f99-4337-9725-1152f46a6b59" />
Поступающие данные в мониторинге Скриншот 3
<img width="1913" height="989" alt="Данные поступающие на агентов в Мониторинге" src="https://github.com/user-attachments/assets/0cdc8b45-7a7f-4739-989a-83ebef59c9f8" />
