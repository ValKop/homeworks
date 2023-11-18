# Домашнее задание к занятию «Кластеризация и балансировка нагрузки» - Копанев Валентин

### Задание 1
- Запустите два simple python сервера на своей виртуальной машине на разных портах
- Установите и настройте HAProxy, воспользуйтесь материалами к лекции по [ссылке](2/)
- Настройте балансировку Round-robin на 4 уровне.
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.

### Решение 1

В файл haproxy.cfg добавляем 
```
listen web_tcp
        bind :1325

        server s1 127.0.0.1:8888 check inter 3s
        server s2 127.0.0.1:9999 check inter 3s
```
И теперь при запросе на 1325 порт, будет roundrobin балансировка по протоколу tcp (4 уровень)

Скриншот запросов по этому порту:  

![image](https://github.com/ValKop/homeworks/assets/60344304/e1e20b9c-7a76-4cbf-a597-fbb43af034a2)


### Задание 2
- Запустите три simple python сервера на своей виртуальной машине на разных портах
- Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4
- HAproxy должен балансировать только тот http-трафик, который адресован домену example.local
- На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.

### Решение 2
В файл haproxy.cfg добавляем 
```
frontend example
        mode http
        bind :8088
        acl ACL_example.local hdr(host) -i example.local
        use_backend web_servers if ACL_example.local

backend web_servers
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check weight 2
        server s2 127.0.0.1:9999 check weight 3
        server s3 127.0.0.1:7777 check weight 4
```

Теперь haproxy слушает порт 8088, с которого проксирует трафик на 1 из 3 серверов только если url подходит под фильтр (example.local). Вот запрос без заголовка example.local:  

![image](https://github.com/ValKop/homeworks/assets/60344304/4e00f4e5-fc20-4202-8862-97dd6500ed8a)


А вот если указать заголовок:  

![image](https://github.com/ValKop/homeworks/assets/60344304/fb2f9daa-552a-450a-800a-7954f51bedf2)
