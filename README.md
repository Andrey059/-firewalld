# Домашнее задание
Сценарии iptables

Описание/Пошаговая инструкция выполнения домашнего задания:  
- Реализовать knocking port
- CentralRouter может попасть на ssh inetrRouter через knock скрипт пример в материалах.
- Добавить inetRouter2, который виден(маршрутизируется (host-only тип сети для виртуалки)) с хоста или форвардится порт через локалхост.
- Запустить nginx на centralServer.
- Пробросить 80й порт на inetRouter2 8080.
- Дефолт в инет оставить через inetRouter. Формат сдачи ДЗ - vagrant + ansible
# Выполнение
Во вложении файлы для проверки ДЗ.


Для проверки нужно будет запустить Vagrantfile и подключиться к centralRouter

```
vagrant ssh centralRouter
```
С centralRouter нужно выполнить следущую команду:

```
cd /vagrant/
/vagrant/knock.sh 192.168.255.1 8881 7777 9991
```

После этого можно подключиться к inetRouter

```
ssh 192.168.255.1
Are you sure you want to continue connecting (yes/no)? yes
```

Пароль: vagrant

В браузере проверить доступность ``` http://192.168.11.171:8080/ ```

С centralRouter нужно выполнить ``` traceroute ya.ru ```
