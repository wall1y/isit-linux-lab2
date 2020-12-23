# isit-linux-lab2

Прежде чем начать. Забыл везде прописывать, что после изменения конфига нужно его сохранять. Это можно делать клавишами `ctrl+O` и затем закрыть файл клавишами `ctrl+X`
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
Тут мы настраиваем два интерфейса, первый, enp0s3 у нас смотрит в интернет, его мы настраиваем на автоматическое получение ip-адреса.
Второй интерфейс, enp0s8 - смотрит в нашу будущую локальную сеть

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

`enp0s3` и `enp0s8` - имена интерфейсов. Они могут у вас быть другими. 

Адреса интерфейса и сети (10.3.0.1 и 10.3.0.0) выберите другие. 192.168.100.1 и 192.168.100.0, например. 

Рекомендуется использовать подсети 192.168.X.Y, 10.0.X.Y, 172.16.X.Y

Вместо X - подставьте любые числа. от 0 до 255. 

Вместо Y - 1 для адреса интерфейса и 0 для адреса сети.

Второго интерфейса может не быть вообще в этом файле, но он будет отображаться в выводе команды `ip a`.

![image](https://user-images.githubusercontent.com/65608414/102983722-544bd480-452e-11eb-8929-bc9754d122f5.png)

После настройки интерфейса устанавливаем DHCP-сервер:

 `apt install isc-dhcp-server`
 
 После установки он попытается запуститься и сломается. Не беда. Сейчас починим.
 
 На всякий случай останавливаем сервер
 `systemctl stop isc-dhcp-server`
 
 Удаляем .pid файл:
 
 ` rm -rf /var/run/dhcpd.pid`
 
 Открываем настройки DHCP сервера:
 
`nano /etc/dhcp/dhcpd.conf`

Приводим его к примерно такому виду, как здесь:
https://github.com/wall1y/isit-linux-lab2/blob/main/dhcpd.conf

Ключевой момент вот здесь:
```
subnet 10.3.0.0 netmask 255.255.255.0 {
  range 10.3.0.10 10.3.0.250;
  option domain-name-servers 8.8.8.8;
  option routers 10.3.0.1;
  option broadcast-address 10.3.0.254;
  default-lease-time 600;
  max-lease-time 7200;
}
```
Меняем циферки, на те, что прописывали при настройках интерфейса выше.

`subnet` - адрес сети, который мы придумали в настройках интерфейса

`range` - даипазон ip адресов, которые будет разадавать наш DHCP сервер. 

`option-routers` - ip-адрес интерфейса, который мы придумали

`option broadcast-address` - широковещательный адрес нашей сети. Берем адрес сети и меняем последний 0 на 254

Помимо этого, отредактируем файл 

`nano /etc/default/isc-dhcp-server`

Закомментируем строку (поставим решётку в начале) `INTERFACESv6=""` и в строку `INTERFACESv4=""` напишем внутри кавычек имя нашего внутренного интерфейса, который смотрит в локальную сеть. У меня оно выглядит вот так: `INTERFACESv4="enp0s8"` у вас возможно иначе. 

После того, как выполнили эти настройки, можем попробовать запустить сервер. 

`systemctl start isc-dhcp-server`

Если всё хорошо, то сервер немного подумает и ничего не скажет нам. В ином случае он скажет что не удалось запустить службу. 

Чтобы узнать причину фиаско, можно ввести команду:

`journalctl -xe`

Выход из неё осуществляется нажатием клавиши `Q`

Ну, если всё получилось, то идём дальше. Настроим возможность пересылки пакетов между интерфейсами. Открываем файл.

`nano /etc/sysctl.conf`

Находим там строку `net.ipv4.ip_forward=1` и раскоментируем её (убираем решётку в начале строки). Если такой строки нет, то пишем её сами.

Применяем настройки командой `sysctl -p`

Далее устанавливаем nftables

`apt install nftables`

После установки открываем файл 
`nano /etc/nftables.conf`

Приводим его к такому же состоянию, как здесь: https://github.com/wall1y/isit-linux-lab2/blob/main/nftables.conf

Обратите внимание, что там используются имена интерфейсов `enp0s3` и `enp0s8` поменяйте их на свои.

После внесения изменений на всякий случай перезапустим службу

`systemctl restart nftables`

Далее установим и запустим веб-сервер:

`apt install apache2`
`systemctl restart apache2`

Теперь пришло время проверить чего мы там наделали. Тут два варианта. Либо мы создаем новую виртуальную машину, указываем ей единственный сетевой адаптер - внутренняя сеть. И дальше всё понятно будет. Либо берем уже существующую виртуалку, например, из первой лабы и в настройках у неё меняем интерфейс на "Внутренняя сеть"

В конце концов, если всё сделано правильно, то мы сможем подключиться к веб-серверу по адресу локального интерфейса, открыв его в браузере на второй виртуальной машине. И НЕ сможем подключиться из внешней сети, пройдя по адресу внешнего интерфейса в браузере на вашем компе, на котором вы всё это делаете. 

Также, если всё сделано правильно, мы сможем подключаться к серверу по ssh и снаружи, по адресу внешнего интерфейса и изнутри, по адресу локального интерфейса. 
Для подключения из локальной сети, введите в терминале  на второй виртуалке следующую команду: `ssh username@ipaddr`, где username - имя пользователя на сервере, а ipaddr - айпи адрес сервера. 

Если всё правильно, система спросит нас, добавить ли этот сервер в доверенные, пишем yes и далее вводим пароль от учетки, которую мы используем для подключения. 
