# Домашнее задание к занятию «Disaster recovery и Keepalived» - Копанев Валентин

### Задание 1
- Дана [схема](1/hsrp_advanced.pkt) для Cisco Packet Tracer, рассматриваемая в лекции.
- На данной схеме уже настроено отслеживание интерфейсов маршрутизаторов Gi0/1 (для нулевой группы)
- Необходимо аналогично настроить отслеживание состояния интерфейсов Gi0/0 (для первой группы).
- Для проверки корректности настройки, разорвите один из кабелей между одним из маршрутизаторов и Switch0 и запустите ping между PC0 и Server0.
- На проверку отправьте получившуюся схему в формате pkt и скриншот, где виден процесс настройки маршрутизатора.

### Решение 1

pkt файл я прикрепил к ДЗ в нетологии. Настройка: в Router1-2 к интерфейску Gig0/1 я добавил `standby 1 track Gig0/0`.  

![image](https://github.com/ValKop/homeworks/assets/60344304/b1ae2d3c-2880-4048-b293-802ea02af61f)

### Задание 2
- Запустите две виртуальные машины Linux, установите и настройте сервис Keepalived как в лекции, используя пример конфигурационного [файла](1/keepalived-simple.conf).
- Настройте любой веб-сервер (например, nginx или simple python server) на двух виртуальных машинах
- Напишите Bash-скрипт, который будет проверять доступность порта данного веб-сервера и существование файла index.html в root-директории данного веб-сервера.
- Настройте Keepalived так, чтобы он запускал данный скрипт каждые 3 секунды и переносил виртуальный IP на другой сервер, если bash-скрипт завершался с кодом, отличным от нуля (то есть порт веб-сервера был недоступен или отсутствовал index.html). Используйте для этого секцию vrrp_script
- На проверку отправьте получившейся bash-скрипт и конфигурационный файл keepalived, а также скриншот с демонстрацией переезда плавающего ip на другой сервер в случае недоступности порта или файла index.html

### Решение 2

Две виртуалки: nginx-x (192.168.152.199) и nginx-y (192.168.152.212).

keepalived.conf NODE X: 
```
vrrp_script check_nginx {
    script "/etc/keepalived/check_nginx.sh"
    interval 3
    weight 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens33
    virtual_router_id 15
    priority 198
    advert_int 1
    virtual_ipaddress {
        192.168.152.200
    }
    track_script {
        check_nginx
    }
}
```

keepalived.conf NODE Y:  
```
vrrp_instance VI_1 {
        state BACKUP
        interface ens33
        virtual_router_id 15
        priority 199
        advert_int 1

        virtual_ipaddress {
              192.168.152.200
        }

}
```

Скрипт на ноде x, проверяющий доступность порта и файла:   
```
#!/bin/bash

nc -z localhost 80
if [ $? -ne 0 ]; then
  exit 1
fi

if [ ! -f /var/www/index.html ]; then
  exit 1
fi

exit 0
```


У ноды x меньший приоритет, хоть она и master, но когда скрипт на ноде x выдает код 0, то ноде добавляется 2 еденицы приоритета и она становится 200. 

Вот когда nginx запущен:  

![image](https://github.com/ValKop/homeworks/assets/60344304/aa9893f7-90f5-4029-a83a-da33e545c1e3)


Когда выключен:  

![image](https://github.com/ValKop/homeworks/assets/60344304/015c48d7-311b-40cf-8812-b1641170edcd)

Лог keepalived, по которому видно, что нода была мастером, но после получения единицы от скрипта, понизила приоритет. Нашла ноду с большим приоритетом и перешла в бэкап стейт.    

![image](https://github.com/ValKop/homeworks/assets/60344304/dab81792-7bca-4361-9ece-97989d6d0b1c)

