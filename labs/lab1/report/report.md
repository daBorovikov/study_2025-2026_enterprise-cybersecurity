---
## Front matter
title: "Отчёт по лабораторной работе №1"
subtitle: "Дисциплина: Кибербезопасность предприятия"
author: 	
	- Боровиков Даниил Александрович,
	- Хрусталев Влад Николаевич,
	- Гисматуллин Артем,
	- Тщесноков Артёмий Pavlovich,
	- Коннова Татьяна,
	- Нефедова Наталья,
	- Уткина Алина,
	- Бансимба Клодели
## Generic otions
lang: ru-RU
toc-title: "Содержание"

## Bibliography
bibliography: bib/cite.bib
csl: pandoc/csl/gost-r-7-0-5-2008-numeric.csl

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
polyglossia-otherlangs:
  name: english
## I18n babel
babel-lang: russian
babel-otherlangs: english
## Fonts
mainfont: Arial
romanfont: Arial
sansfont: Arial
monofont: Arial
mainfontoptions: Ligatures=TeX
romanfontoptions: Ligatures=TeX
sansfontoptions: Ligatures=TeX,Scale=MatchLowercase
monofontoptions: Scale=MatchLowercase,Scale=0.9
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




#  Задание

Сценарий №2

Защита контроллера домена предприятия

Внешний злоумышленник находит в интернете сайт Компании и решает провести атаку на него с целью получения доступа к внутренним ресурсам компании. Обнаружив несколько уязвимостей на внешнем периметре и закрепившись на одном из серверов, Злоумышленник проводит разведку корпоративной сети с целью захватить контроллер домена.
Квалификация нарушителя средняя. Он умеет использовать инструментарий для проведения атак, а также знает техники постэксплуатации.
Злоумышленник обладает опытом проведения почтовых фишинговых рассылок.


# Выполнение лабораторной работы

##  Способы детектирования атаки

###  Детектирование с ViPNet IDS NS

- SQL-инъекция: Сканирование, Blind SQL-Injection, загрузка файла  (рис. [-@fig:001]) (рис. [-@fig:002]).  (рис. [-@fig:003]). 
- Зафиксировали инцидент на платформе (рис. [-@fig:004]). 

- RDP Brute-force: Множественные подключения (рис. [-@fig:005]).  
- Зафиксировали инцидент на платформе (рис. [-@fig:006]). 
- Зафиксировали инцидент на платформе (рис. [-@fig:024])


![Сканирование на SQL-инъекции](image/1.jpg){ #fig:001 width=100% }



![Детектирование SQL-инъекции](image/2.jpg){ #fig:002 width=100% }



![Загрузка вредоносного файла](image/3.jpg){ #fig:003 width=100% }


![Инцидент атака на веб сервер](image/4.jpg){ #fig:004 width=100% }




![RDP Brute-force](image/5.jpg){ #fig:005 width=100% }

 

![Инцидент атака а хост, Brute-force ](image/6.jpg){ #fig:006 width=100% }




![Инцидент Атака на Administration WS](image/24.jpg){ #fig:024 width=100% }






##  Перечень уязвимостей и последствий

Мы выявили и устранили три уязвимости и три последствия:

1. Уязвимость 1: SQL-инъекция.
2. Последствие: Web portal meterpreter.
3. Уязвимость 2: Отключенная защита антивируса.
4. Последствие: Admin meterpreter.
5. Уязвимость 3: Слабый пароль учетной записи.
6. Последствие: Добавление привилегированного пользователя.

###  SQL-инъекция

На узле Web Server PHP (порт 80) была уязвимость в веб-сервисе. Нарушитель использовал sqlmap для загрузки PHP reverse shell (рис. [-@fig:007]).

 
![PHP reverse shell](image/7.jpg){ #fig:007 width=100% }




**Устранение:** Параметр `$id` в GET-запросе проверяли на тип с помощью `is_numeric()`. Изменили функцию `actionView()` в NewsController.php (рис. [-@fig:008]) (рис. [-@fig:009]) (рис. [-@fig:010])





![Поиск места уязвимого параметра](image/8.jpg){ #fig:008 width=100% }



![Измененная функция actionView](image/9.jpg){ #fig:009 width=100% }



![Удаление вредоносного файла](image/10.jpg){ #fig:010 width=100% }


После изменений уязвимость устранена.

###  Последствие: Web portal meterpreter


Нарушитель установил shell-сессию. Мы проверили сокеты командой `ss -tp`  (рис. [-@fig:011]) и завершили сессию: `sudo ss -K dst HACKER_IP dport=HACKER_PORT`
(рис. [-@fig:012]).






![Список установленных соединений](image/11.jpg){ #fig:011 width=100% }


![Завершение сессий](image/12.jpg){ #fig:012 width=100% }





Сессии завершены.

###  Отключенная защита антивируса

На Administrator Workstation отключена реал-тайм защита Windows Defender, что позволило запустить diag.ps1.  Удалили запись в реестре: `REG DELETE "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /v DisableAntiSpyware` (рис. [-@fig:020]) Перезапустили Virus & Threat Protection и включили Real-time Protection (рис. [-@fig:013]) Перезагрузили систему.





![Удаление записи DisableAntiSpyware](image/20.jpg){ #fig:020 width=100% }









![Включение Real-time Protection](image/13.jpg){ #fig:013 width=100% }





###  Последствие: Admin meterpreter

Сессия обнаружена `netstat -ano` (рис. [-@fig:019]) Завершили: `taskkill /f /pid <PID>` (рис. [-@fig:018])



![Соединение с машиной нарушителя](image/19.jpg){ #fig:019 width=100% }


![Остановка процесса](image/18.jpg){ #fig:018 width=100% }



Сессия завершена.

### Слабый пароль учетной записи

На MS Active Directory слабый пароль администратора позволил brute-force по RDP (код события 1149). (рис. [-@fig:017]) Изменили пароль: `net user Administrator *` (рис. [-@fig:016])



![Логи подключений по RDP](image/17.jpg){ #fig:017 width=100% }





![ Изменение пароля](image/16.jpg){ #fig:016 width=100% }

Уязвимость устранена.

###  Последствие: AD User

Добавление пользователя "Hacked" отслежено в Event Viewer (ID 4720, Рисунок 16). Удалили в Active Directory Users and Computers (рис. [-@fig:014]) (рис. [-@fig:015])











![Лог добавления нового пользователя](image/14.jpg){ #fig:014 width=100% }



![Удаление пользователя](image/15.jpg){ #fig:015 width=100% }




Пользователь удален.



Уязвимости устранены, мы научились находить инциденты несанкционированного доступа, находить последствия и исправлять уязвимости и их последствия на контролируемой системе(рис. [-@fig:023])

![Итоговый результат](image/23.jpg){ #fig:023 width=100% }



# Выводы

В рамках учебно-практического занятия на базе программного комплекса обучения методам обнаружения, анализа и устранения последствий компьютерных атак «Ampire» мы выполнили сценарий №2 «Защита контроллера домена предприятия». 

Внешний злоумышленник находит в интернете сайт Компании и решает провести атаку на него с целью получения доступа к внутренним ресурсам компании. Обнаружив несколько уязвимостей на внешнем периметре и закрепившись на одном из серверов, злоумышленник проводит разведку корпоративной сети с целью захватить контроллер домена.

Квалификация нарушителя средняя. Он умеет использовать инструментарий для проведения атак, а также знает техники постэксплуатации. Злоумышленник обладает опытом проведения почтовых фишинговых рассылок.

Уровень сложности сценария — 7 (из 10). Мы успешно выявили уязвимости, проанализировали последствия атаки, устранили их и отработали методы детектирования с использованием инструментов ViPNet IDS NS, ViPNet TIAS и Security Onion.
