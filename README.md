# System Design социальной сети для путешественников
Проект в рамках [курса по System Design](https://balun.courses/courses/system_design) 

### Функциональные требования
 - публикация постов из путешествий с фотографиями, небольшим описанием и привязкой к конкретному месту путешествия;
 - оценка и комментарии постов других путешественников;
 - подписка на других путешественников, чтобы следить за их активностью;
 - поиск популярных мест для путешествий и просмотр постов с этих мест;
 - просмотр ленты других путешественников;

### Нефункциональные требования
 - DAU: 10 000 000;
 - Аудитория: СНГ;
 - Доступность: 99.95%;
 - Сезонность: есть, в период отпусков нагрузка возрастает в 30 раз;
 - Хранение данных: вечное;
 - Активность:
   - просматривает в ленте 30 постов в день;
   - ставит 10 реакций в день;
   - просматривает 100 комментариев в день;
   - оставляет 5 комментариев в день;
   - в сезон отпусков:
     - создает 2 поста с 5 фото каждый в день;
   - в остальное время:
     - 4 поста с 3 фото каждый в месяц;

 - Лимиты:
   - до 10 постов в день;
   - до 300 символов в описании;
   - до 100 комментариев в день;
   - до 300 символов в комментарии;
   - до 5 фото на пост;
   - размер фото до 2Mb;
   - до 1 000 подписок;
   - до 1 000 постов в ленте;
 - Тайминги:
   - добавление поста - до 1с;
   - подгрузка порции постов из ленты - до 3с;
   - загрузка подборки популярных мест - до 2с;

### Оценка нагрузки

Connections = 10 000 000 * 0.1 = 1 000 000

#### Посты
###### Медиа (средний размер фото = 1Mb):
- **write**
  - сезон: 
    - RPS = 10 000 000 * 2 / 86400 = 231
    - Traffic = 231 * 5Mb = 1.2 Gb/s
  - не сезон:
    - RPS = 10 000 000 / 7.5 / 86400 = 15
    - Traffic = 15 * 3Mb = 45 Mb/s
- **read**
  - сезон:
    - RPS = 10 000 000 * 30 / 86400 = 3472
    - Traffic = 3472 * 5Mb = 17.36 Gb/s
  - не сезон:
    - RPS = 10 000 000 * 30 / 86400 = 3472
    - Traffic = 3472 * 3Mb = 10.4 Gb/s
###### Метаинформация (id(16b) + author_id(16b) + body(300 * 4b) + created_at(8b) = 1240b):
- **write**
  - сезон: 
    - RPS = 10 000 000 * 2 / 86400 = 231
    - Traffic = 231 * 1.24Kb = 286.44 Kb/s
  - не сезон:
    - RPS = 10 000 000 / 7.5 / 86400 = 15
    - Traffic = 15 * 1.24Kb = 18.6 Kb/s
- **read**
  - RPS = 10 000 000 * 30 / 86400 = 3472
  - Traffic = 3472 * 1.24Kb = 4.31 Mb/s
   
#### Реакции
В среднем у поста 10 реакций
 - **write** (id (16b) + post_id (16b) + reaction_type(1b) + created_at(8b) = 40b)
   - RPS = 10 000 000 * 10 / 86400 = 1157
   - Traffic = 1157 * 40b = 46.28 Kb/s
 - **read** (post_id(16b) + (reaction_type(1b) + amount(3b)) * 10 = 56b)
   - RPS = 10 000 000 * 30 / 86400 = 3472
   - Traffic = 3472 * 56b = 194.432 Kb/s

#### Комментарии
id(16b) + post_id(16b) + author_id(16b) + body(300b) + created_at(8b) = 356b
 - **write** 
   - RPS = 10 000 000 * 5 / 86400 = 579
   - Traffic = 579 * 356b = 206.124 Kb/s
 - **read**
   - RPS = 10 000 000 * 100 / 86400 = 11574
   - Traffic = 11574 * 356b = 4.12 Mb/s