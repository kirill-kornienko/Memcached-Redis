# Домашнее задание к занятию «Уязвимости и атаки на информационные системы» - `Корниенко Кирилл`


### Задание 1. 

### Кейс
Скачайте и установите виртуальную машину Metasploitable: [https://sourceforge.net/projects/metasploitable/](https://sourceforge.net/projects/metasploitable/)

Это типовая ОС для экспериментов в области информационной безопасности, с которой следует начать при анализе уязвимостей.

Просканируйте эту виртуальную машину, используя nmap.

Сами уязвимости можно поискать на сайте [https://www.exploit-db.com/](https://www.exploit-db.com/)

Для этого нужно в поиске ввести название сетевой службы, обнаруженной на атакуемой машине, и выбрать подходящие по версии уязвимости.

Ответьте на следующие вопросы:

- Какие сетевые службы в ней разрешены?

- Какие уязвимости были вами обнаружены? (список со ссылками: достаточно трёх уязвимостей)

*Приведите ответ в свободной форме.*

### Ответ:

- Какие сетевые службы в ней разрешены?

![nmap](https://github.com/kirill-kornienko/Backup/blob/main/nmap.png)

- Какие уязвимости были вами обнаружены? (список со ссылками: достаточно трёх уязвимостей)

1. vsftpd 2.3.4 - [Backdoor Command Execution (Metasploit)](https://www.exploit-db.com/exploits/17491)

2. Samba 3.0.20  - ['Username' map script' Command Execution (Metasploit)](https://www.exploit-db.com/exploits/16320)

3. UnrealIRCd 3.2.8.1 - [Backdoor Command Execution (Metasploit)](https://www.exploit-db.com/exploits/16922)

### Задание 2

Проведите сканирование Metasploitable в режимах SYN, FIN, Xmas, UDP.

Запишите сеансы сканирования в Wireshark.

Ответьте на следующие вопросы:

- Чем отличаются эти режимы сканирования с точки зрения сетевого трафика?

- Как отвечает сервер?

*Приведите ответ в свободной форме.*

### Ответ:

#### Режим SYN

Отправляется пакет с флагом SYN для установки соединения. Ответ SYN/ACK - порт открыт (После происходит сброс соединения RST). Ответ RST/ACK - порт закрыт. 

![nmap_sS](https://github.com/kirill-kornienko/Security/blob/main/nmap_sS.png)

![wireahark_sS](https://github.com/kirill-kornienko/Security/blob/main/wireshark_syn.png)

#### Режим FIN

Отправляется пакет с флагом FIN. Ответ RST/ACK - порт закрыт. Если ответа нет - порт открыт|фильтруется.

![nmap_sF](https://github.com/kirill-kornienko/Security/blob/main/nmap_sF.png)

![wireshark_sF](https://github.com/kirill-kornienko/Security/blob/main/wireshark_fin.png)

#### Режим Xmas

Отправляется пакет с флагами FIN/PSH/URG. Ответ RST/ACK - порт закрыт. Если ответа нет - порт открыт|фильтруется.

![nmap_sX](https://github.com/kirill-kornienko/Security/blob/main/nmap_sX.png)

![wireshark_sX](https://github.com/kirill-kornienko/Security/blob/main/wireshark_xmas.png)

#### Режим UDP

Отправляет пустой UDP заголовок на каждый порт. Ответ ICMP ошибка о недостижимости порта (тип 3, код 3) - порт закрыт. Другие ICMP ошибки недостижимости - порт фильтруется. После нескольких попыток без ответа - порт открыт|фильтруется. Ответ UDP - порт открыт.

![nmap_udp](https://github.com/kirill-kornienko/Security/blob/main/nmap_udp.png)

![wireshark_udp](https://github.com/kirill-kornienko/Security/blob/main/wireshark_udp.png)



