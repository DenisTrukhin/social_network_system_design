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

### Оценка дисков
#### Посты
###### Медиа
Traffic = 1.2 + 17.36 = 18560 Mb/s <br>
IOPS = 231 + 3472 = 3703 <br>
Capacity = 1.2 Gb * 86400 * 365 = 37843.2 Tb
- **HDD**
  - Disks_for_capacity = 37843.2 / 32 = 1182.6
  - Disks_for_throughput = 18560 / 100 = 185.6
  - Disks_for_iops = 3703 / 100 = 37.03
  - Disks = max(ceil(1182.6), ceil(185.6), ceil(37.03)) = 1183
- **SSD (SATA)**
  - Disks_for_capacity = 37843.2 / 100 = 378.4
  - Disks_for_throughput = 18560 / 500 = 37.1
  - Disks_for_iops = 3703 / 1000 = 3.7
  - Disks = max(ceil(378.4), ceil(37.1), ceil(3.7)) = 379 
- **SSD (NVMe)**
  - Disks_for_capacity = 37843.2 / 30 = 1261.4
  - Disks_for_throughput = 18.56 / 3 = 6.2 
  - Disks_for_iops = 3703 / 10000 = 0.37
  - Disks = max(ceil(1261.4), ceil(6.2), ceil(0.37)) = 1262

Итог: для хранения медиа постов оптимальным будет выбор SSD (SATA) (379 x 100Tb)

###### Метаинформация
Traffic = 286.44 Kb/s + 4.31 Mb/s = 4.6 Mb/s  <br>
IOPS = 231 + 3472 = 3703 <br>
Capacity = 286.44 Kb * 86400 * 365 = 9.03 Tb
- **HDD**
  - Disks_for_capacity = 9.03 / 3  = 3.01
  - Disks_for_throughput = 4.6 / 100 = 0.05
  - Disks_for_iops = 3703 / 100 = 37.03
  - Disks = max(ceil(3.01), ceil(0.04), ceil(37.03)) = 38
- **SSD (SATA)**
  - Disks_for_capacity = 9.03 / 3 = 3.01
  - Disks_for_throughput = 4.6 / 500 = 0.01
  - Disks_for_iops = 3703 / 1000 = 3.7
  - Disks = max(ceil(3.01), ceil(0.01), ceil(3.7)) = 4
- **SSD (NVMe)**
  - Disks_for_capacity = 9.03 / 10 = 0.9
  - Disks_for_throughput = 4.6 / 3000 = 0.002
  - Disks_for_iops = 3703 / 10000 = 0.37
  - Disks = max(ceil(0.9), ceil(0.002), ceil(0.37)) = 1

Итог: для хранения метаинформации постов оптимальным будет выбор SSD (SATA) (4 x 3Tb)

#### Реакции
Traffic = 46.28 + 194.432 = 240.7 Kb/s <br>
IOPS = 1157 + 3472 = 4629 <br>
Capacity = 46.28 * 86400 * 365 = 1.5 Tb
- **HDD**
  - Disks_for_capacity = 1.5 / 2 = 0.75
  - Disks_for_throughput = 240.7 Kb/s / 100 Mb/s = 0.002
  - Disks_for_iops = 4629 / 100 = 46.3
  - Disks = max(ceil(0.75), ceil(0.002), ceil(46.3)) = 47
- **SSD (SATA)**
  - Disks_for_capacity = 1.5 / 2 = 0.75
  - Disks_for_throughput = 240.7 Kb/s / 500 Mb/s = 0.0004
  - Disks_for_iops = 4629 / 1000 = 4.6
  - Disks = max(ceil(0.75), ceil(0.0004), ceil(4.6)) = 5
- **SSD (NVMe)**
  - Disks_for_capacity = 1.5 / 2 = 0.75
  - Disks_for_throughput = 240.7 Kb/s / 3 Gb/s =  0.00008
  - Disks_for_iops = 4629 / 10000 = 0.46 
  - Disks = max(ceil(0.75), ceil(0.00008), ceil(0.46)) = 1

Итог: для хранения реакций оптимальным будет выбор SSD (NVMe) (1 x 2Tb)

#### Комментарии
Traffic = 206.124 Kb/s + 4.12 Mb/s = 4.33 Mb/s <br>
IOPS = 579 + 11574 = 12153 <br>
Capacity = 206.124 Kb * 86400 * 365 = 6.5 Tb
- **HDD**
  - Disks_for_capacity = 6.5 / 2 = 3.25
  - Disks_for_throughput = 4.33 Mb/s / 100 = 0.04 
  - Disks_for_iops = 12153 / 100 = 121.53
  - Disks = max(ceil(3.25), ceil(0.04), ceil(121.53)) = 122
- **SSD (SATA)**
  - Disks_for_capacity = 6.5 / 2 = 3.25
  - Disks_for_throughput = 4.33 Mb/s / 500 = 0.009
  - Disks_for_iops = 12153 / 1000 = 12.15
  - Disks = max(ceil(3.25), ceil(0.009), ceil(12.15)) = 13
- **SSD (NVMe)**
  - Disks_for_capacity = 6.5 / 4 = 1.63
  - Disks_for_throughput = 4.33 Mb/s / 3 Gb/s = 0.001
  - Disks_for_iops = 12153 / 10000 = 1.22
  - Disks = max(ceil(1.63), ceil(0.001), ceil(1.22)) = 2

Итог: для хранения комментариев оптимальным будет выбор SSD (NVMe) (2 x 4Tb)

### Оценка хостов

#### Посты
###### Медиа
 - Disks = 379 (SSD SATA 100Tb)
 - Disks_per_host = 2
 - Replication_factor = 2
 - Hosts = 379 / 2 = 190
 - Hosts_with_replication = 190 * 2 = 380

###### Метаинформация
 - Disks = 4 (SSD SATA 3Tb)
 - Disks_per_host = 1
 - Replication_factor = 3
 - Hosts = 4 / 1 = 4
 - Hosts_with_replication = 4 * 3 = 12 

#### Реакции
 - Disks = 1 (SSD NVMe 2Tb)
 - Disks_per_host = 1
 - Replication_factor = 3 
 - Hosts = 1 / 1 = 1
 - Hosts_with_replication = 1 * 3 = 3 

#### Комментарии
 - Disks = 2 (SSD NVMe 4Tb)
 - Disks_per_host = 1
 - Replication_factor = 3 
 - Hosts = 2 / 1 = 2
 - Hosts_with_replication = 2 * 3 = 6 