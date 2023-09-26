# Discord

## 1. Тема и целевая аудитория

### Ключевой функционал
- Логин/Регистрация
- Личные и групповые голосовые чаты
- Отправка сообщений с вложениями
- "Сервера" с несколькими голосовыми и текстовыми каналами

### Целевая аудитория
150 млн пользователей в месяц по всему миру  
- Северная Америка - 30% 
- Европа - 15%  
- Южная Америка - 12%

Остальное распределено по миру.

## 2. Расчет нагрузки

### Продуктовые метрики

| Метрика                                      | Значение |
| -------------------------------------------- | -------- |
| Зарегистрированных пользователей             | 500 млн  |
| Месячная аудитория                           | 150 млн  |
| Суточная аудитория                           | 20 млн   |
| Максимальное количество пользователей онлайн | 10 млн   |
| Минут голосового общения в день              | 4 млрд   |
| Сообщений в день                             | 4 млрд   |
| "Серверов" и групповых чатов                 | 25 млн   |

### Количество операций по типам

Количество сообщений на пользователя в день - `4 млрд / 20 млн = 200`  
Будем считать, что 5% отправляемых пользователем сообщений имеют вложения

Считаем, что пользователь меняет свою аватарку/аватарку сервера раз в две недели

| Операция                         | Среднее кол-во в день на пользователя |
| -------------------------------- | :-----------------------------------: |
| Аутентификация                   |                   2                   |
| Смена аватара                    |                 0.07                  |
| Вход в голосовой чат/канал       |                  15                   |
| Получение списка участников чата |                  40                   |
| Получение списка сообщений чата  |                  40                   |
| Получение списка чатов           |                  20                   |
| Отправка сообщения               |                  200                  |
| Отправка вложения                |                  10                   |

### Технические метрики

#### Хранилище

Хранилище требуется для пользовательских данных, данных серверов/чатов и хранения сообщений.    
Данными серверов/чатов и пользователей, кроме аватаров можно принебречь.
Размер аватара в среднем равен 2 Кб (отображаются в клиенте максимум размером 96*96)  

Имеем:  
`2 Кб * (500 млн + 25 млн) = 1 Тб` 

Среднюю длину сообщения, отправленного пользователем, будем считать равным 100 символов,
а размер символа - 2 байтам. Т.е. средний размер сообщения - 200 байт.

Средний размер вложения считаем равным 1 Мб.  
Будем считать, что 5% отправляемых пользователем сообщений имеют вложения

Discord хранит около 4 триллионов сообщений, для их хранения потребуется:  
`4e12 * (200 + 0.05 * 1000) / 1e12 = 1000 Тб`

| Тип данных | Размер  |
| ---------- | ------- |
| Аватары    | 1 Тб    |
| Сообщения  | 1000 Тб |
 
#### RPS

Учитывая 20 млн DAU и 10 млн пиковый онлайн:  
Считаем средний RPS по формуле: `20 млн * N / 24 / 3600`, где N - число операций определённого типа в день  
Считаем пиковый RPS, как 2 * средний RPS

| Операция                         | Средний RPS | Пиковый RPS |
| -------------------------------- | :---------: | :---------: |
| Аутентификация                   |     463     |     926     |
| Смена аватара                    |     16      |     32      |
| Вход в голосовой чат/канал       |    3471     |    6942     |
| Получение списка участников чата |    9256     |   18 512    |
| Получение списка сообщений чата  |    9256     |   18 512    |
| Получение списка чатов           |    4628     |    9256     |
| Отправка сообщения               |   46 296    |   92 592    |
| Отправка вложения                |    2314     |    4628     |

#### Сетевой трафик

**Голосовые чаты**

Пусть битрейт звука в голосовых чатах - 96 кбит/с  

Учитывая 4 млрд минут голосового общения в день, суммарный суточный трафик для голосовых чатов:  
`4 млрд * 60 с * 96 кбит/с = 23 040 000 Гбит/сутки = 23 040 000 * 0.116 = 2 672 640 Гбайт/сутки`

Учитывая 20 млн DAU, среднее потребление трафика для голосовых чатов в секунду:  
`23 040 000 Гбит/сутки / 24 / 3600 = 266.7 Гбит/c`

Пиковое потребление трафика считаем равным 2 * среднее:
`2 * 266.7 = 533.4 Гбит/c`

Для голосовых чатов исходящий трафик равен входящему, т.к. входящие голосовые потоки микшируются на сервере в 1 исходящий.

**Смена аватара**  
Относится к входящему трафику, средний размер аватара при отправке считаем равным 500 Кб.

**Получение списка сообщений**  
Относится к исходящему трафику  
Считаем, что на каждый запрос пользователь получает последние 15 сообщений,
при среднем размере в 200 байт и среднем размере вложения в 1 Мб на один запрос имеем:  
`15 * (200 + 0.05 * 1e6) = 0.753 Мб`

**Получение списка чатов**  
Относится к исходящему трафику  
Считаем, что на каждый запрос пользователь получает информацию о 20 чатах/"серверах",
при среднем размере аватара в 2 Кб имеем:
`20 * 2 = 40 Кб`

Получение списка участников чата считаем аналогично.

**Отправка сообщения/вложения**  
При получении сообщения от пользователя его надо отправить, в среднем, 5 другим пользователям  
Таким образом, исходящий трафик в 5 раз больше входящего

**Входящий трафик**
| Операция           | Средний трафик, Гбит/с | Пиковый трафик, Гбит/с | Суточный трафик, Гбайт |
| ------------------ | :--------------------: | :--------------------: | :--------------------: |
| Голосовой чат      |         266.7          |         533.4          |       2 672 640        |
| Смена аватара      |          0.06          |          0.12          |          648           |
| Отправка сообщения |          0.07          |          0.14          |          756           |
| Отправка вложения  |           18           |           36           |        194 400         |

**Исходящий трафик**
| Операция                         | Средний трафик, Гбит/с | Пиковый трафик, Гбит/с | Суточный трафик, Гбайт |
| -------------------------------- | :--------------------: | :--------------------: | :--------------------: |
| Голосовой чат                    |         266.7          |         533.4          |       2 672 640        |
| Получение списка участников чата |           3            |           6            |         32 400         |
| Получение списка сообщений чата  |          55.8          |         111.6          |        602 640         |
| Получение списка чатов           |          1.5           |           3            |         16 200         |
| Отправка сообщения               |          0.35          |          0.7           |         3 780          |
| Отправка вложения                |           90           |          180           |        972 000         |

## 3. Глобальная балансировка нагрузки

### Физическое расположение датацентров

| Регион           | Процент трафика | Процент датацентров |
| ---------------- | :-------------: | :-----------------: |
| Северная Америка |       35        |         50          |
| Европа           |       25        |         30          |
| Азия             |       15        |         20          |

В каждом регионе предлагаю распределить датацентры относительно плотности пользователей, учитывая:

1. В США 30% трафика => большинство датацентров Северной Америки расположим там
2. Discord заблокирован в Китае => не учитываем при распределении датацентров по Азии

### Схема балансировки

1. Определяем регион через Geo-based DNS
2. В рамках региона выбираем датацентр через BGP Anycast

Если речь идёт о голосовых чатах, выбор датацентра аналогичен для создателя чата.  
Остальным участникам передаём IP датацентра, чтобы убедиться, что трафик от всех участников чата обрабатывается одним датацентром.

##  Источники
1. https://discord.com/company
2. https://www.bankmycell.com/blog/number-of-discord-users
3. https://worldpopulationreview.com/country-rankings/discord-users-by-country
4. https://www.semrush.com/website/discord.com/overview/
5. https://support.discord.com/hc/en-us/articles/11635925354775-Audio-Bitrate-FAQ
6. https://www.statista.com/statistics/1349341/discord-messages-sent-per-day/
7. https://startupgeek.com/blog/discord-usage-and-stats-2023
