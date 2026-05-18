
Как приложение был поднят образ _containous/whoami_, который выводит данные о запросе, ну и соответственно хэдеры.

## Тесты
### 1) Полная цепь всех Nginx

Попытаемся заспуфить хэдэр X-Forwarded-For с помощью утилиты curl:
``` bash
curl -H "X-Forwarded-For: 99.99.99.99" http://localhost:8081/
```


Видим, что nginx перетер подмену хэдэра и вывел всю цепь, где _**172.18.0.1**_ - адрес виртуального сетевого шлюза.
```
```bash
Hostname: e135898b73f9
IP: 127.0.0.1
IP: ::1
IP: 172.18.0.2
RemoteAddr: 172.18.0.3:42542
GET / HTTP/1.1
Host: localhost
User-Agent: curl/8.19.0
Accept: */*
X-Forwarded-For: 172.18.0.1, 172.18.0.5, 172.18.0.4
```

### 2) Обход балансировщика

Попытаемся заспуфить хэдэр X-Forwarded-For с помощью утилиты curl, обращаясь ко второму nginx:
``` bash
curl -H "X-Forwarded-For: 99.99.99.99" http://localhost:8082/
```


Видим, что nginx увидел, что запрос пришел от IP - адреса, который не находиться в доверенной подсети, и перетер его
```
Hostname: e135898b73f9
IP: 127.0.0.1
IP: ::1
IP: 172.18.0.2
RemoteAddr: 172.18.0.3:34226
GET / HTTP/1.1
Host: localhost
User-Agent: curl/8.19.0
Accept: */*
X-Forwarded-For: 172.18.0.1, 172.18.0.4
```

### 3) Запрос напрямую к приложению

Попытаемся заспуфить хэдэр X-Forwarded-For с помощью утилиты curl, обращаясь ко второму nginx:
``` bash
curl -H "X-Forwarded-For: 99.99.99.99" http://localhost:8082/
```


Отработал корректно: сбросил фейковый заголовок и передал в приложение только чистый адрес источника запроса
```
Hostname: e135898b73f9
IP: 127.0.0.1
IP: ::1
IP: 172.18.0.2
RemoteAddr: 172.18.0.3:57686
GET / HTTP/1.1
Host: localhost
User-Agent: curl/8.19.0
Accept: */*
X-Forwarded-For: 172.18.0.1
```
