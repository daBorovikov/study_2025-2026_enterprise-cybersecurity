---
## Front matter
lang: ru-RU
title: Лабораторная работа №3
subtitle: Кибербезопасность предприятия
author:
  - Боровиков Даниил
  - Хрусталев Влад 
  - Гисматуллин Артём
  - Чесноков Артёмий
  - Коннова Татьяна
  - Нефедова Наталья
  - Уткина Алина
  - Бансимба Клодели

institute:
  - Российский университет дружбы народов, Москва, Россия

## i18n babel
babel-lang: russian
babel-otherlangs: english

## Formatting pdf
toc: false
toc-title: Содержание
slide_level: 2
aspectratio: 169
section-titles: true
theme: metropolis
header-includes:
 - \metroset{progressbar=frametitle,sectionpage=progressbar,numbering=fraction}
 - '\makeatletter'
 - '\beamer@ignorenonframefalse'
 - '\makeatother'
 
## Fonts
mainfont: PT Serif
romanfont: PT Serif
sansfont: PT Sans
monofont: PT Mono
mainfontoptions: Ligatures=TeX
romanfontoptions: Ligatures=TeX
sansfontoptions: Ligatures=TeX,Scale=MatchLowercase
monofontoptions: Scale=MatchLowercase,Scale=0.9
---

## Задачи

В рамках лабораторной работы проведен анализ трех уязвимых узлов с различными типами уязвимостей. Использовались инструменты мониторинга и защиты: ViPNet IDS NS, Security Onion, 
а также встроенные средства операционных систем для обнаружения и нейтрализации атак.

## Используемые инструменты

- **ViPNet IDS NS** - обнаружение сетевых атак
- **Security Onion** - мониторинг сетевой безопасности
- **Системные утилиты** (ss, netstat, taskkill) - анализ процессов и соединений

# Процесс выполнения лабораторной работы

# Уязвимый узел: WpDiscuz

## Уязвимый узел: WpDiscuz

![Вредоносные файлы](image/01.png){ #fig:001 width=70%, height=70% }

## Уязвимый узел: WpDiscuz

![Активная версия плагина](image/02.png){ #fig:002 width=70%, height=70% }

## Уязвимый узел: WpDiscuz

![Восстановление из резервной копии](image/03.png){ #fig:003 width=70%, height=70% }

## Уязвимый узел: WpDiscuz

![Восстановление из резервной копии](image/012.png){ #fig:012 width=70%, height=70% }

## Уязвимый узел: WpDiscuz

![Восстановление из резервной копии](image/027.png){ #fig:027 width=70%, height=70% }

## Уязвимый узел: WpDiscuz

![Завершение вредоносных соединений](image/04.png){ #fig:004 width=70%, height=70% }

## Проверка устранения

- Отсутствие активных соединений с хостом нарушителя
- Восстановлена оригинальная версия сайта
- Плагин деактивирован или обновлен

# Уязвимый узел: RocketChat

## Уязвимый узел: RocketChat

![Несанкционированные подключения](image/021.png){ #fig:021 width=70%, height=70% }

## Уязвимый узел: RocketChat

![Создание backdoor-соединений 2](image/016.png){ #fig:016 width=70%, height=70% }

## Уязвимый узел: RocketChat

![Сброс пароля администратора](image/09.png){ #fig:009 width=70%, height=70% }

## Уязвимый узел: RocketChat

![Сброс пароля администратора 2](image/013.png){ #fig:013 width=70%, height=70% }

## Уязвимый узел: RocketChat

![Настройка](image/011.png){ #fig:011 width=70%, height=70% }

## Уязвимый узел: RocketChat

![Настройка](image/014.png){ #fig:014 width=70%, height=70% }

## Уязвимый узел: RocketChat

![Настройка](image/018.png){ #fig:018 width=70%, height=70% }

## Уязвимый узел: RocketChat

![Отключение выполнения JavaScript в MongoDB](image/015.png){ #fig:015 width=70%, height=70% }

## Уязвимый узел: RocketChat

![Восстановление базы данных](image/019.png){ #fig:019 width=70%, height=70% }

## Проверка устранения

- Успешная аутентификация с двухфакторной проверкой
- Отсутствие вредоносных соединений
- Корректная работа всех функций RocketChat

# Уязвимый узел: Proxylogon

## Уязвимый узел: Proxylogon

![Установление meterpreter-сессий](image/07.png){ #fig:007 width=70%, height=70% }

## Уязвимый узел: Proxylogon

![Блокировка доступа](image/05.png){ #fig:005 width=70%, height=70% }

## Уязвимый узел: Proxylogon

![Блокировка доступа](image/06.png){ #fig:006 width=70%, height=70% }

## Уязвимый узел: Proxylogon

![Завершение вредоносных процессов](image/08.png){ #fig:008 width=70%, height=70% }

## Проверка устранения

- Отсутствие несанкционированных подключений
- Невозможность доступа к /ecp извне
- Отсутствие вредоносных файлов и процессов

## Выводы

В ходе лабораторной работы успешно проанализированы и нейтрализованы уязвимости трех различных веб-приложений:

1. **WpDiscuz** - уязвимость типа RCE через загрузку файлов
2. **RocketChat** - комбинация NoSQL-инъекции и уязвимости ядра ОС  
3. **Proxylogon** - комплексная атака на Exchange Server

## Завершение работы

![Выполнение работы](image/030.png){ #fig:030 width=70%, height=70% }

## Завершение работы

![Заполнение инцидентов](image/031.png){ #fig:031 width=70%, height=70% }
