# -Zabbix-PostgreSQL
# Установка и настройка системы мониторинга Zabbix

Проект содержит пошаговую установку и настройку системы мониторинга **Zabbix** с использованием **PostgreSQL**, **Apache2**, **PHP 8.3**, веб-интерфейса и агентов. Также реализована авторегистрация узлов и добавление новых хостов для мониторинга.

## Содержание

- [Описание](#описание)
- [Установка компонентов](#установка-компонентов)
  - [PostgreSQL](#установка-сервера-postgresql)
  - [Apache2](#установка-apache2)
  - [PHP 8.3](#установка-php)
  - [Zabbix Server](#настройка-и-запуск-zabbix-сервера)
  - [Веб-интерфейс Zabbix](#установка-веб-интерфейса-zabbix)
  - [Zabbix Agent](#установка-клиента-zabbix)
- [Добавление хоста](#добавление-нового-хоста-на-сервер-zabbix)
- [Авторегистрация узлов](#авторегистрация-узлов)
- [Файлы](#файлы)
- [Доступ по умолчанию](#доступ-по-умолчанию)

## Описание

Zabbix — это мощная система мониторинга состояния сетевых сервисов, серверов, оборудования и виртуальных машин. Веб-интерфейс позволяет управлять конфигурациями, собирать метрики и отслеживать инциденты.

---

## Установка компонентов

### Установка сервера PostgreSQL

```bash
apt-get install postgresql17-server zabbix-server-pgsql fping
/etc/init.d/postgresql initdb
systemctl enable --now postgresql
su - postgres -s /bin/sh -c 'createuser --pwprompt zabbix'
su - postgres -s /bin/sh -c 'createdb -O zabbix zabbix'
systemctl restart postgresql
```
Импорт схемы базы данных:

bash
```
psql -U zabbix -f /usr/share/doc/zabbix-common-database-pgsql-*/schema.sql zabbix
psql -U zabbix -f /usr/share/doc/zabbix-common-database-pgsql-*/images.sql zabbix
psql -U zabbix -f /usr/share/doc/zabbix-common-database-pgsql-*/data.sql zabbix
```
Установка Apache2
bash
```
apt-get install apache2 apache2-mod_php8.3
systemctl enable --now httpd2
```
Установка PHP
bash
```
apt-get install php8.3 php8.3-mbstring php8.3-sockets php8.3-gd php8.3-xmlreader php8.3-pgsql php8.3-ldap php8.3-openssl
```
Изменения в /etc/php/8.3/apache2-mod_php/php.ini:
```
memory_limit = 256M
post_max_size = 32M
max_execution_time = 600
max_input_time = 600
date.timezone = Europe/Moscow
always_populate_raw_post_data = -1
bash
```
systemctl restart httpd2
Настройка и запуск Zabbix-сервера
Файл: /etc/zabbix/zabbix_server.conf
```
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=<ВАШ_ПАРОЛЬ>
```
bash
```
systemctl enable --now zabbix_pgsql
```
Установка веб-интерфейса Zabbix
bash
```
apt-get install zabbix-phpfrontend-apache2 zabbix-phpfrontend-php8.3
ln -s /etc/httpd2/conf/addon.d/A.zabbix.conf /etc/httpd2/conf/extra-enabled/
systemctl restart httpd2
chown apache2:apache2 /var/www/webapps/zabbix/ui/conf
Перейдите в браузере по адресу:
http://<ip-сервера>/zabbix
```
Если возникает ошибка "доступ запрещен", добавьте Require all granted в файл /etc/httpd2/conf/sites-available/default.conf внутри секции <Directory>, и перезапустите Apache:

bash
```
systemctl restart httpd2
```
Установка клиента Zabbix
bash
```
apt-get install zabbix-agent
```
Файл конфигурации агента: /etc/zabbix/zabbix_agentd.conf

```
Server=<ip-сервера>
ServerActive=<ip-сервера>
Hostname=comp01.example.test
```
bash
```
systemctl enable --now zabbix_agentd.service
```
Добавление нового хоста на сервер Zabbix
В веб-интерфейсе:

Перейдите в Сбор данных → Узлы сети

Нажмите Создать узел сети

Укажите:

Имя узла

IP-адрес

Шаблон: Linux by Zabbix agent

Группу

Нажмите Добавить

Авторегистрация узлов
Можно настроить автоматическую регистрацию агентов через Настройки → Действия → Автоматическая регистрация. Это позволит автоматически добавлять хосты в зависимости от имени, IP или используемого шаблона.


Доступ по умолчанию
После установки Zabbix:

Логин: Admin
Пароль: zabbix
