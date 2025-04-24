# Домашнее задание к занятию 2 "`Кластеризация и балансировка нагрузки`" - `Лебедев Виктор`


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

---

### Задание 1

1. `Запустите два simple python сервера на своей виртуальной машине на разных портах`
2. `Установите и настройте HAProxy, воспользуйтесь материалами к лекции по ссылке`
3. `Настройте балансировку Round-robin на 4 уровне.`
4. `На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy.`

### Решение 1

**1. Создание и запуск Python серверов**
```
python3 -m http.server 8888 --bind 0.0.0.0
python3 -m http.server 9999 --bind 0.0.0.0
```
**2. Установка и настройка HAProxy**
```
sudo apt update
sudo apt install haproxy
sudo nano /etc/haproxy/haproxy.cfg
```
**3. Добавляем в haproxy.cfg**
```
listen stats  # веб-страница со статистикой
        bind                    :888
        mode                    http
        stats                   enable
        stats uri               /stats
        stats refresh           5s
        stats realm             Haproxy\ Statistics

frontend example  # секция фронтенд
        mode tcp
        bind :8088
        default_backend web_servers

backend web_servers    # секция бэкенд
        mode tcp
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:8888 check
        server s2 127.0.0.1:9999 check
```
**4. Скриншот работы балансировки**
<img src = "img\img1.jpg" width = 100%>
**5. Конфиг HAProxy**
<a href="haproxy1.cfg">Конфиг к заданию 1</a>

---

### Задание 2

1. `Запустите три simple python сервера на своей виртуальной машине на разных портах`
2. `Настройте балансировку Weighted Round Robin на 7 уровне, чтобы первый сервер имел вес 2, второй - 3, а третий - 4`
3. `HAproxy должен балансировать только тот http-трафик, который адресован домену example.local`
4. `На проверку направьте конфигурационный файл haproxy, скриншоты, где видно перенаправление запросов на разные серверы при обращении к HAProxy c использованием домена example.local и без него.`

### Решение 2

**1. Создание и запуск Python серверов**
```
python3 -m http.server 7777 --bind 0.0.0.0
python3 -m http.server 8888 --bind 0.0.0.0
python3 -m http.server 9999 --bind 0.0.0.0
```
**2. Вносим изменения в haproxy.cfg**
```
frontend example  # секция фронтенд
        mode http
        bind :8088
        acl ACL_example.com hdr(host) -i example.com
        use_backend web_servers if ACL_example.com

backend web_servers    # секция бэкенд
        mode http
        balance roundrobin
        option httpchk
        http-check send meth GET uri /index.html
        server s1 127.0.0.1:7777 weight 2 check
        server s2 127.0.0.1:8888 weight 3 check
        server s3 127.0.0.1:9999 weight 4 check
```
**3. Скриншот работы балансировки**
<img src = "img\img2.jpg" width = 100%>
**4. Конфиг HAProxy**
<a href="haproxy2.cfg">Конфиг к заданию 2</a>
