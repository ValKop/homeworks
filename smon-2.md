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

Вводим пароль от БД в кофиг /etc/zabbix/zabbix-server.conf и вручаем сервисам автозапуск после перезагрузки:  
`systemctl restart zabbix-server zabbix-agent apache2`   
`systemctl enable zabbix-server zabbix-agent apache2`

Скриншоты:  
![image](https://github.com/ValKop/homeworks/assets/60344304/4bb86322-c0ef-4940-bd50-76567d257a82)  
![image](https://github.com/ValKop/homeworks/assets/60344304/074008f8-6768-4e53-94df-c9b1da34db08)


### Задание 2

Установите Zabbix Agent на два хоста.

### Решение 2

Загрузим и установим deb архив с данными о репозитории заббикса:

`wget https://repo.zabbix.com/zabbix/6.4/debian/pool/main/z/zabbix-release/zabbix-release_6.4-1+debian11_all.deb`  
`dpkg -i zabbix-release_6.4-1+debian11_all.deb`

Устанавливаем агент:  
`apt install zabbix-agent`

Вручаем сервису автозапуск после перезагрузки:  
`systemctl restart zabbix-agent`   
`systemctl enable zabbix-agent`

В /etc/zabbix/zabbix-agentd.conf прописываем IP ZBX-Сервера и в вебне добавляем хост на мониторинг, навесив шаблон (чтобы не париться, самый популярный - Linux by Zabbix Agent).

Хосты подключены к серверу:  
![image](https://github.com/ValKop/homeworks/assets/60344304/1398eb27-a85c-4710-b772-c3c156906ae8)

Latest data:  
![image](https://github.com/ValKop/homeworks/assets/60344304/f1c3245b-b5e5-4899-ac02-d0b8db50ce93)

Лог:  
![image](https://github.com/ValKop/homeworks/assets/60344304/b2ca546d-b149-48b1-96df-c16a275dbcc6)
