# Домашнее задание к занятию "Система мониторинга Zabbix" - Копанев Валентин

### Задание 1

Установите Zabbix Server с веб-интерфейсом

### Решение 1

Загрузим и установим deb архив с данными о репозитории заббикса:

`wget https://repo.zabbix.com/zabbix/6.4/debian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb`  
`dpkg -i zabbix-release_6.4-1+debian11_all.deb`

Устанавливаем сервер, агент, фронт, апач, пыху, постгрес и компоненты заббикса, для работы с этим:

`apt install zabbix-server-pgsql zabbix-agent zabbix-frontend-php apache2 php7.4-pgsql postgresql zabbix-apache-conf zabbix-sql-scripts`

В постгресе создаём пользователя и базу данных, которой он будет владеть:  
`sudo -u postgres createuser --pwprompt zabbix`  
`sudo -u postgres createdb -O zabbix zabbix`

Импортируем начальную схему и данные:  
`zcat /usr/share/zabbix-sql-scripts/postgresql/server.sql.gz | sudo -u zabbix psql zabbix`





`systemctl restart zabbix-server zabbix-agent apache2`  
`systemctl enable zabbix-server zabbix-agent apache2`
