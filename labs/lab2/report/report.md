---
## Front matter
title: "Отчет по лабораторной работе №2"
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

# Задание. Сценарий №5. 

ЗАЩИТА НАУЧНО-ТЕХНИЧЕСКОЙ ИНФОРМАЦИИ ПРЕДПРИЯТИЯ

Внешний нарушитель умеет использовать инструментарий для
проведения компьютерных атак, знает техники постэксплуатации.

Средство обнаружения вторжений – программно-аппаратный комплекс
для обнаружения вторжений в информационные системы ViPNet IDS NS.

Автоматическое выявление инцидентов на основе интеллектуального
анализа событий информационной безопасности – программно-аппаратный
комплекс ViPNet TIAS.

ViPNet EPP применяется для защиты отдельных компонентов
информационной инфраструктуры организаций – персональных компьютеров
пользователей и корпоративных серверов.

Специализированный контроль сетевой безопасности и предотвращение
проникновения, упрощающие централизованное управление сетью – Security
Onion.
Security Onion связывает воедино три основные функции:
 - полный захват пакетов;
 - обнаружение сетей и конечных точек;
 - мощные инструменты анализа.
 
# Последовательность действий нарушителя


1. Внутренний нарушитель подбирает пароль на файловый сервер и
меняет существующий на сервере файл другим файлом с backdoor (дефектом
алгоритма).

2. Пользователь Dev-1 загружает и запускает файл с backdoor.

3. Внутренний нарушитель получает контроль над компьютером
пользователя Dev-1 и загружает скрипт для похищения учетных данных из
браузера. Запускает данный скрипт и получает логин и пароль к Redmine.

4. Внутренний нарушитель проводит атаку stored XSS для включения
на Redmine сервере REST API. Вредоносный код записывается на Wikiстраницу проекта Dev1. Получив доступ к консоли администратора,
внутренний нарушитель создает нового пользователя Redmine с правами
администратора.

5. Внутренний нарушитель ожидает, когда администратор
просмотрит страницу с внедренным вредоносным кодом.

6. Внутренний нарушитель проводит Blind SQL-инъекцию, получает
доступ к данным конфиденциального проекта.

# Выполнение лабораторной работы

## Перечень уязвимостей и последствий

### Слабый пароль пользователя

**Обнаружение:** На файловом сервере использовался слабый пароль учетной записи dev1.

**Устранение:** Пароль изменен на сложный через Active Directory Users and Computers.

### Последствие Dev backdoor

**Обнаружение:**

 - В папке Downloads пользователя dev1 обнаружен файл `svchosting.exe`
 - В планировщике задач создано задание `Evil task` с автозапуском при входе пользователя
 - Задание настроено на выполнение каждые 5 минут
 
**Устранение:**

 - Удалено задание `Evil task` из планировщика задач
 - Удален файл svchosting.exe из папки Downloads (рис. [-@fig:001] - рис. [-@fig:003])

![Файл svchosting.exe в папке Downloads](image/01.PNG){ #fig:001 width=100% height=100% }

![Задание "Evil task" в планировщике задач](image/02.PNG){ #fig:002 width=100% height=100% }

![Настройки задания с путем к вредоносному файлу](image/03.PNG){ #fig:003 width=100% height=100% }

### Уязвимость 2: XSS (CVE-2019-17427)

**Обнаружение:** На Wiki-странице проекта DEV1 обнаружен сложный XSS-код, который:

 - Создает пользователя "hacker" с правами администратора
 - Включает REST API в настройках Redmine
 - Использует событие onfocusin для автоматического выполнения

**Устранение:**

 - В файле redcloth3.rb исправлена обработка HTML-тегов
 - Удален тег pre из списка разрешенных тегов (ALLOWED_TAGS)
 - Перезапущена служба nginx: sudo systemctl restart nginx.service (рис. [-@fig:004] - рис. [-@fig:007])

![Вредоносный XSS-код на Wiki-странице](image/04.PNG){ #fig:004 width=100% height=100% }

![Исходный код файла redcloth3.rb](image/05.PNG){ #fig:005 width=100% height=100% }

![Исправленная версия файла redcloth3.rb](image/06.PNG){ #fig:006 width=100% height=100% }

![Перезапуск службы nginx](image/07.PNG){ #fig:007 width=100% height=100% }

### Последствие: Redmine User

**Обнаружение:** В Redmine создан пользователь "hacker" с email hacker@hacker.ru (рис. [-@fig:008])

**Устранение:** Пользователь "hacker" удален через веб-интерфейс Redmine (рис. [-@fig:009])

![Список пользователей Redmine с пользователем "hacker"](image/08.PNG){ #fig:008 width=100% height=100% }

![Подтверждение удаления пользователя](image/09.PNG){ #fig:009 width=100% height=100% }

### Уязвимость 3: Blind SQL-инъекция (CVE-2019018890)

**Обнаружение:**

 - В логах Redmine зафиксированы запросы с SLEEP(3) в параметре subproject_id

 - Время выполнения запросов увеличилось до 6074 мс (вместо обычных ~30 мс) (рис. [-@fig:010])

**Устранение:**

 - В файле query.rb закомментирован уязвимый код обработки subproject_id

 - Добавлена фильтрация входных параметров (рис. [-@fig:011])

![Логи Redmine с SQL-инъекцией](image/010.PNG){ #fig:010 width=100% height=100% }

![Исправление в файле query.rb](image/011.PNG){ #fig:011 width=100% height=100% }

Общий результат выполненной работы: (рис. [-@fig:012] - рис. [-@fig:013])

![Главная страница](image/012.PNG){ #fig:012 width=100% height=100% }

![Карточки инцидентов и последствий](image/013.PNG){ #fig:013 width=100% height=100% }


# Общие выводы

В рамках учебно-практического занятия на базе программного комплекса обучения методам обнаружения, анализа и устранения последствий компьютерных атак «Ampire» мы выполнили сценарий №5 «Защита научно-технической информации предприятия». Внутренний нарушитель, используя слабые пароли и уязвимости в веб-приложении Redmine, осуществил комплексную атаку с целью получения доступа к конфиденциальной информации. 
Нарушитель применил техники внедрения backdoor, эксплуатации XSS-уязвимости и слепой SQL-инъекции для создания привилегированного пользователя и несанкционированного доступа к данным. Уровень сложности сценария — 8 (из 10). Мы успешно выявили уязвимости, проанализировали последствия атаки, устранили их и отработали методы детектирования с использованием инструментов ViPNet IDS NS, ViPNet TIAS и Security Onion, 
а также освоили методики исправления исходного кода приложений для устранения уязвимостей.


