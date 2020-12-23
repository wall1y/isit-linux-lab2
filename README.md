# isit-linux-lab2
В первую очередь настроим виртуальную машину. Необходимо будет добавить второй сетевой интерфейс.
Для первого интерфейса выбираем "Сетевой мост", для второго - "Внутренняя сеть" и вводим имя сети.

Адаптер1, он будет у нас внешним интерфейсом:


![image](https://user-images.githubusercontent.com/65608414/102971455-3fb21100-451b-11eb-889b-dc1ed0423122.png)

Адаптер2, он будет у нас локальным интерфейсом:

Имя сети можно ввести любое. 

![image](https://user-images.githubusercontent.com/65608414/102971610-7daf3500-451b-11eb-9401-fc9b91d4d033.png)

После этого устанавливаем виртуальную машину для сервера. Тут я думаю, вы справитесь. Во время установки дебиан спросит, какой интерфейс использовать, для установки - выбирайте первый. 

После установки сервера, логинимся, становимся рутом

`su -`

Затем устанавливаем SSH, он понадобится для проверки доступности порта (в задании указано) и для более удобного настраивания сервера. 

`apt install ssh`

Далее идем по ссылке https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.74-installer.msi
скачиваем программу и запускаем её.

![image](https://user-images.githubusercontent.com/65608414/102981279-8c511880-452a-11eb-9000-490943efa1ed.png)

Узнаём ip-адрес сервера:
`ip a`

![image](https://user-images.githubusercontent.com/65608414/102982037-b48d4700-452b-11eb-89e2-7bc9c4203037.png)

Вводим ip-адрес в Host Nsme (or IP address) и нажимаем open.

![image](https://user-images.githubusercontent.com/65608414/102982141-e30b2200-452b-11eb-8f00-00d7c29e8484.png)

После того, как мы подключились, снова становимся рутом

`su -`

Настраиваем интерфейсы на нашем сервере.
Открываем файл с настройками:

`nano /etc/network/interfaces`

Редактируем его содержимое, чтобы оно выглядело как-то так:

```
source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback

allow-hotplug enp0s3
iface enp0s3 inet dhcp

allow-hotplug enp0s8
iface enp0s8 inet static
address 10.3.0.1
netmask 255.255.255.0
network 10.3.0.0
broadcast 10.3.0.255
```


