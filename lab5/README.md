# Практическая работа 5
tanya.maksimova.2004@yandex.ru

## Название

Исследование информации о состоянии беспроводных сетей

## Цель работы

1.  Получить знания о методах исследования радиоэлектронной обстановки

2.  Составить представление о механизмах работы Wi-Fi сетей на канальном
    и сетевом уровне модели OSI

3.  Закрепить практические навыки использования языка программирования R
    для обработки данных

4.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

## Исходные данные

1.  Ноутбук с ОС Windows 10
2.  Rstudio Desktop
3.  Интерпретатор R 4.5.1

``` r
sessionInfo()
```

    R version 4.5.1 (2025-06-13 ucrt)
    Platform: x86_64-w64-mingw32/x64
    Running under: Windows 10 x64 (build 19045)

    Matrix products: default
      LAPACK version 3.12.1

    locale:
    [1] LC_COLLATE=Russian_Russia.utf8  LC_CTYPE=Russian_Russia.utf8   
    [3] LC_MONETARY=Russian_Russia.utf8 LC_NUMERIC=C                   
    [5] LC_TIME=Russian_Russia.utf8    

    time zone: Europe/Moscow
    tzcode source: internal

    attached base packages:
    [1] stats     graphics  grDevices utils     datasets  methods   base     

    loaded via a namespace (and not attached):
     [1] compiler_4.5.1    fastmap_1.2.0     cli_3.6.5         tools_4.5.1      
     [5] htmltools_0.5.9   rstudioapi_0.17.1 yaml_2.3.10       rmarkdown_2.30   
     [9] knitr_1.50        jsonlite_2.0.0    xfun_0.54         digest_0.6.37    
    [13] rlang_1.1.6       evaluate_1.0.5   

## Задание

Используя программный пакет dplyr языка программирования R провести
анализ журналов и ответить на вопросы

## Ход работы

1.  Подготовить данные

2.  Провести анализ точек доступа

3.  Провести анализ данных клиентов

## Выполнение шагов

### Шаг 1

Установим и подключим необходимые пакеты:

``` r
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.5.2


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
library(readr)
```

    Warning: пакет 'readr' был собран под R версии 4.5.2

``` r
library(tidyverse)
```

    Warning: пакет 'tidyverse' был собран под R версии 4.5.2

    Warning: пакет 'ggplot2' был собран под R версии 4.5.2

    Warning: пакет 'tibble' был собран под R версии 4.5.2

    Warning: пакет 'tidyr' был собран под R версии 4.5.2

    Warning: пакет 'purrr' был собран под R версии 4.5.2

    Warning: пакет 'forcats' был собран под R версии 4.5.2

    Warning: пакет 'lubridate' был собран под R версии 4.5.2

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ forcats   1.0.1     ✔ stringr   1.5.2
    ✔ ggplot2   4.0.1     ✔ tibble    3.3.0
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.2.0     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

#### 1. Импортируйте данные

``` r
td_data <- read_csv("P2_wifi_data.csv", skip = 1, n_max = 167)
```

    Rows: 167 Columns: 15
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (6): BSSID, Privacy, Cipher, Authentication, LAN IP, ESSID
    dbl  (6): channel, Speed, Power, # beacons, # IV, ID-length
    lgl  (1): Key
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
client_data <- read_csv("P2_wifi_data.csv", skip = 169)
```

    Warning: One or more parsing issues, call `problems()` on your data frame for details,
    e.g.:
      dat <- vroom(...)
      problems(dat)

    Rows: 12081 Columns: 7
    ── Column specification ────────────────────────────────────────────────────────
    Delimiter: ","
    chr  (3): Station MAC, BSSID, Probed ESSIDs
    dbl  (2): Power, # packets
    dttm (2): First time seen, Last time seen

    ℹ Use `spec()` to retrieve the full column specification for this data.
    ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

#### 2. Привести датасеты в вид “аккуратных данных”, преобразовать типы столбцов в соответствии с типом данных

``` r
td_data <- td_data %>% 
    rename(
    first_seen = `First time seen`,
    last_seen = `Last time seen`,
    speed = Speed,
    privacy = Privacy,
    cipher = Cipher,
    auth = Authentication,
    power = Power,
    beacons = `# beacons`,
    iv = `# IV`,
    lan_ip = `LAN IP`,
    id_length = `ID-length`,
    essid = ESSID,
    key = Key
  ) %>% 
  mutate(
    first_seen = as_datetime(first_seen),
    last_seen = as_datetime(last_seen),
    channel = as.integer(channel),
    speed = as.integer(speed),
    power = as.integer(power),
    beacons = as.integer(beacons),
    iv = as.integer(iv),
    id_length = as.integer(id_length))
```

``` r
client_data <- client_data %>%
    rename(
    station_mac = `Station MAC`,
    first_seen = `First time seen`,
    last_seen = `Last time seen`,
    power = Power,
    packets = `# packets`,
    probed_ESSIDs = `Probed ESSIDs`
  ) %>%
  mutate(
    first_seen = as_datetime(first_seen),
    last_seen = as_datetime(last_seen),
    power = as.integer(power),
    packets = as.integer(packets))
```

#### 3. Просмотрите общую структуру данных с помощью функции glimpse()

``` r
glimpse(td_data)
```

    Rows: 167
    Columns: 15
    $ BSSID      <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9:04:1…
    $ first_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 09:13…
    $ last_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 11:53…
    $ channel    <int> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, 6, 6,…
    $ speed      <int> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180, 65, …
    $ privacy    <chr> "WPA2", "WPA2", "WPA2", "WPA2", "WPA2", "OPN", "WPA2", "WPA…
    $ cipher     <chr> "CCMP", "CCMP", "CCMP", "CCMP", "CCMP", NA, "CCMP", "CCMP",…
    $ auth       <chr> "PSK", "PSK", "PSK", "PSK", "PSK", NA, "PSK", "PSK", "PSK",…
    $ power      <int> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42, -62,…
    $ beacons    <int> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 1390, 1…
    $ iv         <int> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, 907, …
    $ lan_ip     <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.  0. …
    $ id_length  <int> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 12, 0,…
    $ essid      <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "MIREA_…
    $ key        <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,…

``` r
glimpse(client_data)
```

    Rows: 12,081
    Columns: 7
    $ station_mac   <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E:1…
    $ first_seen    <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 09…
    $ last_seen     <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 11…
    $ power         <int> -33, -65, -39, -61, -53, -43, -31, -71, -74, -65, -45, -…
    $ packets       <int> 858, 4, 432, 958, 1, 344, 163, 3, 115, 437, 265, 77, 7, …
    $ BSSID         <chr> "BE:F1:71:D5:17:8B", "(not associated)", "BE:F1:71:D6:10…
    $ probed_ESSIDs <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U13…

### Шаг 2

Анализ точек доступа:

#### 1. Определить небезопасные точки доступа (без шифрования – OPN)

``` r
unsave_td <- td_data %>% filter(privacy == "OPN") %>% select(BSSID)
unsave_td
```

    # A tibble: 42 × 1
       BSSID            
       <chr>            
     1 E8:28:C1:DC:B2:52
     2 E8:28:C1:DC:B2:50
     3 E8:28:C1:DC:B2:51
     4 E8:28:C1:DC:FF:F2
     5 00:25:00:FF:94:73
     6 E8:28:C1:DD:04:52
     7 E8:28:C1:DE:74:31
     8 E8:28:C1:DE:74:32
     9 E8:28:C1:DC:C8:32
    10 E8:28:C1:DD:04:50
    # ℹ 32 more rows

#### 2. Определить производителя для каждого обнаруженного устройства

``` r
oui_db <- read.csv("oui.csv")

get_manufacturer <- function(mac) {
  oui <- gsub(":", "", substr(mac, 1, 8))
  result <- oui_db[oui_db$Assignment == oui, ]
  
  if(nrow(result) > 0) {
    return(result$Organization.Name[1])
  } else {
    return(NA)
  }
}

unsave_td <- unsave_td %>% mutate(manufacturer = sapply(BSSID, get_manufacturer))

unsave_td %>% select(BSSID, manufacturer) %>% print(n = Inf)
```

    # A tibble: 42 × 2
       BSSID             manufacturer                
       <chr>             <chr>                       
     1 E8:28:C1:DC:B2:52 Eltex Enterprise Ltd.       
     2 E8:28:C1:DC:B2:50 Eltex Enterprise Ltd.       
     3 E8:28:C1:DC:B2:51 Eltex Enterprise Ltd.       
     4 E8:28:C1:DC:FF:F2 Eltex Enterprise Ltd.       
     5 00:25:00:FF:94:73 Apple, Inc.                 
     6 E8:28:C1:DD:04:52 Eltex Enterprise Ltd.       
     7 E8:28:C1:DE:74:31 Eltex Enterprise Ltd.       
     8 E8:28:C1:DE:74:32 Eltex Enterprise Ltd.       
     9 E8:28:C1:DC:C8:32 Eltex Enterprise Ltd.       
    10 E8:28:C1:DD:04:50 Eltex Enterprise Ltd.       
    11 E8:28:C1:DD:04:51 Eltex Enterprise Ltd.       
    12 E8:28:C1:DC:C8:30 Eltex Enterprise Ltd.       
    13 E8:28:C1:DE:74:30 Eltex Enterprise Ltd.       
    14 E0:D9:E3:48:FF:D2 Eltex Enterprise Ltd.       
    15 E8:28:C1:DC:B2:41 Eltex Enterprise Ltd.       
    16 E8:28:C1:DC:B2:40 Eltex Enterprise Ltd.       
    17 00:26:99:F2:7A:E0 Cisco Systems, Inc          
    18 E8:28:C1:DC:B2:42 Eltex Enterprise Ltd.       
    19 E8:28:C1:DD:04:40 Eltex Enterprise Ltd.       
    20 E8:28:C1:DD:04:41 Eltex Enterprise Ltd.       
    21 E8:28:C1:DE:47:D2 Eltex Enterprise Ltd.       
    22 02:BC:15:7E:D5:DC <NA>                        
    23 E8:28:C1:DC:C6:B1 Eltex Enterprise Ltd.       
    24 E8:28:C1:DD:04:42 Eltex Enterprise Ltd.       
    25 E8:28:C1:DC:C8:31 Eltex Enterprise Ltd.       
    26 E8:28:C1:DE:47:D1 Eltex Enterprise Ltd.       
    27 00:AB:0A:00:10:10 <NA>                        
    28 E8:28:C1:DC:C6:B0 Eltex Enterprise Ltd.       
    29 E8:28:C1:DC:C6:B2 Eltex Enterprise Ltd.       
    30 E8:28:C1:DC:BD:50 Eltex Enterprise Ltd.       
    31 E8:28:C1:DC:0B:B2 Eltex Enterprise Ltd.       
    32 E8:28:C1:DC:33:12 Eltex Enterprise Ltd.       
    33 00:03:7A:1A:03:56 Taiyo Yuden Co., Ltd.       
    34 00:03:7F:12:34:56 Atheros Communications, Inc.
    35 00:3E:1A:5D:14:45 <NA>                        
    36 E0:D9:E3:49:00:B1 Eltex Enterprise Ltd.       
    37 E8:28:C1:DC:BD:52 Eltex Enterprise Ltd.       
    38 00:26:99:F2:7A:EF Cisco Systems, Inc          
    39 02:67:F1:B0:6C:98 <NA>                        
    40 02:CF:8B:87:B4:F9 <NA>                        
    41 00:53:7A:99:98:56 <NA>                        
    42 E8:28:C1:DE:47:D0 Eltex Enterprise Ltd.       

#### 3. Выявить устройства, использующие последнюю версию протокола шифрования WPA3, и названия точек доступа, реализованных на этих устройствах

``` r
td_data %>% filter(str_detect(privacy, "WPA3")) %>% select(BSSID, essid, privacy)
```

    # A tibble: 8 × 3
      BSSID             essid                                          privacy  
      <chr>             <chr>                                          <chr>    
    1 26:20:53:0C:98:E8  <NA>                                          WPA3 WPA2
    2 A2:FE:FF:B8:9B:C9 "Christie’s"                                   WPA3 WPA2
    3 96:FF:FC:91:EF:64  <NA>                                          WPA3 WPA2
    4 CE:48:E7:86:4E:33 "iPhone (Анастасия)"                           WPA3 WPA2
    5 8E:1F:94:96:DA:FD "iPhone (Анастасия)"                           WPA3 WPA2
    6 BE:FD:EF:18:92:44 "Димасик"                                      WPA3 WPA2
    7 3A:DA:00:F9:0C:02 "iPhone XS Max \U0001f98a\U0001f431\U0001f98a" WPA3 WPA2
    8 76:C5:A0:70:08:96  <NA>                                          WPA3 WPA2

#### 4. Отсортировать точки доступа по интервалу времени, в течение которого они находились на связи, по убыванию.

``` r
td_sessions <- td_data %>%
  arrange(BSSID, first_seen) %>%
  group_by(BSSID) %>%
  mutate(gap = as.numeric(difftime(first_seen, lag(last_seen, default = first(first_seen)))), sess_id = cumsum(gap > 2700) + 1) %>%
  group_by(BSSID, sess_id) %>%
  summarise(start = min(first_seen), end = max(last_seen), .groups = 'drop') %>%
  group_by(BSSID) %>%
  mutate(sessions = n()) %>%
  ungroup() %>%
  mutate(duration = as.numeric(end - start, units = "secs")) %>%
  arrange(desc(duration)) %>%
  select(BSSID, sessions, start, end, duration)
td_sessions
```

    # A tibble: 167 × 5
       BSSID             sessions start               end                 duration
       <chr>                <int> <dttm>              <dttm>                 <dbl>
     1 00:25:00:FF:94:73        1 2023-07-28 09:13:06 2023-07-28 11:56:21     9795
     2 E8:28:C1:DD:04:52        1 2023-07-28 09:13:09 2023-07-28 11:56:05     9776
     3 E8:28:C1:DC:B2:52        1 2023-07-28 09:13:03 2023-07-28 11:55:38     9755
     4 08:3A:2F:56:35:FE        1 2023-07-28 09:13:27 2023-07-28 11:55:53     9746
     5 6E:C7:EC:16:DA:1A        1 2023-07-28 09:13:03 2023-07-28 11:55:12     9729
     6 E8:28:C1:DC:B2:50        1 2023-07-28 09:13:06 2023-07-28 11:55:12     9726
     7 48:5B:39:F9:7A:48        1 2023-07-28 09:13:06 2023-07-28 11:55:11     9725
     8 E8:28:C1:DC:B2:51        1 2023-07-28 09:13:06 2023-07-28 11:55:11     9725
     9 E8:28:C1:DC:FF:F2        1 2023-07-28 09:13:06 2023-07-28 11:55:10     9724
    10 8E:55:4A:85:5B:01        1 2023-07-28 09:13:06 2023-07-28 11:55:09     9723
    # ℹ 157 more rows

#### 5. Обнаружить топ-10 самых быстрых точек доступа.

``` r
td_data %>% arrange(desc(speed)) %>% select(BSSID, speed) %>% head(10)
```

    # A tibble: 10 × 2
       BSSID             speed
       <chr>             <int>
     1 26:20:53:0C:98:E8   866
     2 96:FF:FC:91:EF:64   866
     3 CE:48:E7:86:4E:33   866
     4 8E:1F:94:96:DA:FD   866
     5 9A:75:A8:B9:04:1E   360
     6 4A:EC:1E:DB:BF:95   360
     7 56:C5:2B:9F:84:90   360
     8 E8:28:C1:DC:B2:41   360
     9 E8:28:C1:DC:B2:40   360
    10 E8:28:C1:DC:B2:42   360

#### 6. Отсортировать точки доступа по частоте отправки запросов (beacons) в единицу времени по их убыванию

``` r
td_beac <- td_data %>%
  mutate(duration = as.numeric(last_seen - first_seen, units = "secs"), 
         bps = ifelse(duration != 0, beacons / duration, NA)) %>%
  arrange(desc(bps))  %>%
  select(BSSID, essid, first_seen, last_seen, beacons, duration, bps)

td_beac
```

    # A tibble: 167 × 7
       BSSID    essid first_seen          last_seen           beacons duration   bps
       <chr>    <chr> <dttm>              <dttm>                <int>    <dbl> <dbl>
     1 F2:30:A… "iPh… 2023-07-28 10:27:02 2023-07-28 10:27:09       6        7 0.857
     2 B2:CF:C… "Мих… 2023-07-28 10:40:54 2023-07-28 10:40:59       4        5 0.8  
     3 3A:DA:0… "iPh… 2023-07-28 10:27:01 2023-07-28 10:27:10       5        9 0.556
     4 02:BC:1… "MT_… 2023-07-28 09:24:46 2023-07-28 09:24:48       1        2 0.5  
     5 00:3E:1… "MT_… 2023-07-28 10:34:03 2023-07-28 10:34:05       1        2 0.5  
     6 76:C5:A…  <NA> 2023-07-28 11:16:36 2023-07-28 11:16:38       1        2 0.5  
     7 D2:25:9… "Сан… 2023-07-28 09:45:29 2023-07-28 09:45:42       5       13 0.385
     8 BE:F1:7… "C32… 2023-07-28 09:13:03 2023-07-28 11:50:44    1647     9461 0.174
     9 00:03:7… "MT_… 2023-07-28 10:29:13 2023-07-28 10:29:19       1        6 0.167
    10 38:1A:5… "EBF… 2023-07-28 09:13:03 2023-07-28 10:25:02     704     4319 0.163
    # ℹ 157 more rows

### Шаг 3

Анализ данных клиентов:

#### 1. Определить производителя для каждого обнаруженного устройства

``` r
client_data <- client_data %>%
  mutate(manufacturer = sapply(station_mac, get_manufacturer))
client_data %>% select(station_mac, manufacturer) %>% head(10)
```

    # A tibble: 10 × 2
       station_mac       manufacturer                        
       <chr>             <chr>                               
     1 CA:66:3B:8F:56:DD <NA>                                
     2 96:35:2D:3D:85:E6 <NA>                                
     3 5C:3A:45:9E:1A:7B CHONGQING FUGUI ELECTRONICS CO.,LTD.
     4 C0:E4:34:D8:E7:E5 AzureWave Technology Inc.           
     5 5E:8E:A6:5E:34:81 <NA>                                
     6 10:51:07:CB:33:E7 Intel Corporate                     
     7 68:54:5A:40:35:9E Intel Corporate                     
     8 74:4C:A1:70:CE:F7 Liteon Technology Corporation       
     9 8A:A3:5A:33:76:57 <NA>                                
    10 CA:54:C4:8B:B5:3A <NA>                                

#### 2. Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

``` r
client_data %>% filter(!substr(station_mac, 2, 2) %in% c("2", "6", "a", "A", "e", "E")) %>% distinct() %>% select(station_mac)
```

    # A tibble: 220 × 1
       station_mac      
       <chr>            
     1 5C:3A:45:9E:1A:7B
     2 C0:E4:34:D8:E7:E5
     3 10:51:07:CB:33:E7
     4 68:54:5A:40:35:9E
     5 74:4C:A1:70:CE:F7
     6 BC:F1:71:D4:DB:04
     7 4C:44:5B:14:76:E3
     8 A0:E7:0B:AE:D5:44
     9 00:95:69:E7:7F:35
    10 00:95:69:E7:7C:ED
    # ℹ 210 more rows

#### 3. Кластеризовать запросы от устройств к точкам доступа по их именам. Определить время появления устройства в зоне радиовидимости и время выхода его из нее.

``` r
clusters <- client_data %>%
  filter(!is.na(probed_ESSIDs) & probed_ESSIDs != "") %>%
  mutate(networks = strsplit(probed_ESSIDs, ",")) %>%
  unnest(networks) %>%
  mutate(networks = trimws(networks)) %>%
  group_by(station_mac, networks) %>%
  summarise(first = min(first_seen), last = max(last_seen), .groups = "drop") %>%
  mutate(duration = as.numeric(last - first, units = "secs")) %>%
  arrange(station_mac, first)

clusters
```

    # A tibble: 1,831 × 5
       station_mac       networks   first               last                duration
       <chr>             <chr>      <dttm>              <dttm>                 <dbl>
     1 00:90:4C:E6:54:54 "Redmi"    2023-07-28 09:16:59 2023-07-28 10:21:15     3856
     2 00:95:69:E7:7C:ED "nvripcsu… 2023-07-28 09:13:11 2023-07-28 11:56:13     9782
     3 00:95:69:E7:7D:21 "nvripcsu… 2023-07-28 09:13:15 2023-07-28 11:56:17     9782
     4 00:95:69:E7:7F:35 "nvripcsu… 2023-07-28 09:13:11 2023-07-28 11:56:07     9776
     5 00:F4:8D:F7:C5:19 "Hornet24" 2023-07-28 10:45:04 2023-07-28 11:43:26     3502
     6 00:F4:8D:F7:C5:19 "Redmi 12" 2023-07-28 10:45:04 2023-07-28 11:43:26     3502
     7 02:00:00:00:00:00 "CPPK_FRE… 2023-07-28 09:54:40 2023-07-28 11:55:36     7256
     8 02:00:00:00:00:00 "MIREA"    2023-07-28 09:54:40 2023-07-28 11:55:36     7256
     9 02:00:00:00:00:00 "MIREA_HO… 2023-07-28 09:54:40 2023-07-28 11:55:36     7256
    10 02:00:00:00:00:00 "\\xAC\\x… 2023-07-28 09:54:40 2023-07-28 11:55:36     7256
    # ℹ 1,821 more rows

#### 4. Оценить стабильность уровня сигнала внури кластера во времени. Выявить наиболее стабильный кластер.

``` r
clusters %>% inner_join(client_data, by = "station_mac") %>%
filter(first_seen >= first & last_seen <= last) %>% group_by(networks) %>%
summarise(n_requests = n(), mean_power = mean(power, na.rm = TRUE),
sd_power = if(n() == 1) 0 else sd(power, na.rm = TRUE), .groups = "drop") %>%
filter(n_requests >= 5) %>% arrange(sd_power) %>% head(1)
```

    # A tibble: 1 × 4
      networks n_requests mean_power sd_power
      <chr>         <int>      <dbl>    <dbl>
    1 MT_FREE           7      -64.7     2.69

## Вывод

В ходе выполнения 5 практической работы были получены знания о методах
исследования радиоэлектронной обстановки с помощью журналов программных
средств анализа беспроводных сетей.
