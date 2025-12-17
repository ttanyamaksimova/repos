# Практическая работа 4
tanya.maksimova.2004@yandex.ru

## Название

Исследование метаданных DNS трафика

## Цель работы

1.  Закрепить практические навыки использования языка программирования R
    для обработки данных

2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R

3.  Закрепить навыки исследования метаданных DNS трафика

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

Используя программный пакет dplyr, освоить анализ DNS логов с помощью
языка программирования R.

## Ход работы

1.  Импортируйте данные DNS
    https://storage.yandexcloud.net/dataset.ctfsec/dns.zip
2.  Добавьте пропущенные данные о структуре данных (назначении столбцов)
3.  Преобразуйте данные в столбцах в нужный формат
4.  Просмотрите общую структуру данных с помощью функции glimpse()

## Анализ

1.  Сколько участников информационного обмена в сети Доброй Организации?
2.  Какое соотношение участников обмена внутри сети и участников
    обращений к внешним ресурсам?
3.  Найдите топ-10 участников сети, проявляющих наибольшую сетевую
    активность
4.  Найдите топ-10 доменов, к которым обращаются пользователи сети и
    соответственное количество обращений
5.  Опеределите базовые статистические характеристики (функция
    summary()) интервала времени между последовательными обращениями к
    топ-10 доменам
6.  Часто вредоносное программное обеспечение использует DNS канал в
    качестве канала управления, периодически отправляя запросы на
    подконтрольный злоумышленникам DNS сервер. По периодическим запросам
    на один и тот же домен можно выявить скрытый DNS канал. Есть ли
    такие IP адреса в исследуемом датасете?

## Обогащение данных

1.  Определите местоположение (страну, город) и организацию-провайдера
    для топ-10 доменов. Для этого можно использовать сторонние сервисы,
    например http://ip-api.com (API-эндпоинт http://ip-api.com/json)

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
library(httr)
```

    Warning: пакет 'httr' был собран под R версии 4.5.2

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

### Шаг 2

Подготовим данные:

#### 1. Импортируем данные:

``` r
dns_d <- read.csv(file = "dns.log", header = FALSE, sep = '\t', na.strings = c("-"))
```

#### 2. Добавим данные о структуре данных (назначении столбцов):

``` r
col_names <- c("timestamp", "uid", "source_ip", "source_port", "destination_ip", 
  "destination_port", "protocol", "transaction_id", "query", "qclass", 
  "qclass_name", "qtype", "qtype_name", "rcode", "rcode_name", 
  "AA", "TC", "RD", "RA", "Z", "answer", "TTLS", "rejected")
```

``` r
names(dns_d) <- col_names
```

#### 3. Преобразуем данные в столбцах в нужный формат:

``` r
dns_d <- dns_d %>% mutate(timestamp = as_datetime(timestamp), source_port = as.numeric(source_port), qclass = as.numeric(qclass), qtype = as.numeric(qtype))
```

#### 4. Посмотрим общую структуру данных:

``` r
glimpse(dns_d)
```

    Rows: 427,935
    Columns: 23
    $ timestamp        <dttm> 2012-03-16 12:30:05, 2012-03-16 12:30:15, 2012-03-16…
    $ uid              <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jl…
    $ source_ip        <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76"…
    $ source_port      <dbl> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
    $ destination_ip   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255…
    $ destination_port <int> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    $ protocol         <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp…
    $ transaction_id   <int> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 6218…
    $ query            <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass           <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1,…
    $ qclass_name      <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET…
    $ qtype            <dbl> 33, 32, 32, 32, 32, 32, 32, 32, 32, 32, 33, 33, 33, 1…
    $ qtype_name       <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB"…
    $ rcode            <int> 0, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ rcode_name       <chr> "NOERROR", NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…
    $ AA               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ TC               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ RD               <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE…
    $ RA               <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…
    $ Z                <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1,…
    $ answer           <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ TTLS             <chr> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, N…
    $ rejected         <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALS…

### Шаг 3

Анализ:

#### 4. Сколько участников информационного обмена в сети Доброй Организации?

``` r
unique(c(dns_d$source_ip, dns_d$destination_ip)) %>% length()
```

    [1] 1359

#### 5. Какое соотношение участников обмена внутри сети и участников обращений к внешним ресурсам?

``` r
priv_net <- "^(192\\.168\\.|10\\.|172\\.(1[6-9]|2[0-9]|3[0-1])\\.)"
uniq_ip <- unique(c(dns_d$source_ip, dns_d$destination_ip))
inter_ip <- uniq_ip[grepl(priv_net, uniq_ip)]
exter_ip <- uniq_ip[!grepl(priv_net, uniq_ip)]
length(inter_ip)/length(exter_ip)
```

    [1] 13.77174

#### 6. Найдите топ-10 участников сети, проявляющих наибольшую сетевую активность

``` r
dns_d %>% count(source_ip) %>% arrange(desc(n)) %>% head(10)
```

             source_ip     n
    1    10.10.117.210 75943
    2   192.168.202.93 26522
    3  192.168.202.103 18121
    4   192.168.202.76 16978
    5   192.168.202.97 16176
    6  192.168.202.141 14967
    7    10.10.117.209 14222
    8  192.168.202.110 13372
    9   192.168.203.63 12148
    10 192.168.202.106 10784

#### 7. Найдите топ-10 доменов, к которым обращаются пользователи сети и соответственное количество обращений

``` r
dns_d %>% count(query) %>% arrange(desc(n)) %>% head(10)
```

                                                                         query
    1                                                teredo.ipv6.microsoft.com
    2                                                         tools.google.com
    3                                                            www.apple.com
    4                                                           time.apple.com
    5                                          safebrowsing.clients.google.com
    6  *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00
    7                                                                     WPAD
    8                                              44.206.168.192.in-addr.arpa
    9                                                                 HPE8AA67
    10                                                                  ISATAP
           n
    1  39273
    2  14057
    3  13390
    4  13109
    5  11658
    6  10401
    7   9134
    8   7248
    9   6929
    10  6569

#### 8. Опеределите базовые статистические характеристики (функция summary()) интервала времени между последовательными обращениями к топ-10 доменам

``` r
top_10_d <- dns_d %>% count(query) %>% arrange(desc(n)) %>% head(10) %>% pull(query)

for(d in top_10_d) {
  dom_data <- dns_d %>% filter(query == d) %>% arrange(timestamp)
  intervals <- diff(dom_data$timestamp)
  cat("\nДомен:", d, "\n")
  cat("Количество запросов:", nrow(dom_data), "\n")
  print(summary(intervals))}
```


    Домен: teredo.ipv6.microsoft.com 
    Количество запросов: 39273 
    Time differences in secs
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
        0.000     0.000     0.000     2.941     0.510 50387.760 

    Домен: tools.google.com 
    Количество запросов: 14057 
    Time differences in secs
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
        0.000     0.000     0.000     8.187     1.000 50364.830 

    Домен: www.apple.com 
    Количество запросов: 13390 
    Time differences in secs
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
        0.000     0.000     1.000     8.607     3.010 50963.630 

    Домен: time.apple.com 
    Количество запросов: 13109 
    Time differences in secs
         Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
        0.000     0.370     1.760     8.665     4.723 50924.280 

    Домен: safebrowsing.clients.google.com 
    Количество запросов: 11658 
    Time differences in secs
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
        0.00     0.00     1.00    10.00     2.01 49952.32 

    Домен: *\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00 
    Количество запросов: 10401 
    Time differences in secs
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
        0.00     0.15     0.50    11.24     1.50 52723.50 

    Домен: WPAD 
    Количество запросов: 9134 
    Time differences in secs
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
        0.00     0.75     0.75    12.61     1.11 50049.11 

    Домен: 44.206.168.192.in-addr.arpa 
    Количество запросов: 7248 
    Time differences in secs
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
        0.00     2.09     4.00    16.01    20.09 49679.81 

    Домен: HPE8AA67 
    Количество запросов: 6929 
    Time differences in secs
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
        0.00     0.75     0.75    16.61    25.49 50044.43 

    Домен: ISATAP 
    Количество запросов: 6569 
    Time differences in secs
        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
        0.00     0.75     0.76    17.46     1.05 51997.79 

#### 9. Часто вредоносное программное обеспечение использует DNS канал в качестве канала управления, периодически отправляя запросы на подконтрольный злоумышленникам DNS сервер. По периодическим запросам на один и тот же домен можно выявить скрытый DNS канал. Есть ли такие IP адреса в исследуемом датасете?

``` r
susp_ips <- dns_d %>% filter(!is.na(source_ip), !is.na(query), query != "") %>% arrange(source_ip, query, timestamp) %>% group_by(source_ip, query) %>%
  mutate(time_num = as.numeric(timestamp),
    intvl = c(NA, diff(time_num))) %>%
  filter(!is.na(intvl)) %>%
  summarise(n_req = n() + 1, mean_int = mean(intvl), sd_int = sd(intvl),
    .groups = "drop") %>%
  filter(n_req >= 35, mean_int <= 60, sd_int <= 2) %>%
  group_by(source_ip) %>%
  summarise(suspicious_domains = n(), total_requests = sum(n_req),
    avg_interval = mean(mean_int), .groups = "drop") %>%
  arrange(desc(suspicious_domains), desc(total_requests))

susp_ips
```

    # A tibble: 10 × 4
       source_ip       suspicious_domains total_requests avg_interval
       <chr>                        <int>          <dbl>        <dbl>
     1 10.10.117.210                    4            599      0.0592 
     2 192.168.202.103                  2            104      0.00492
     3 192.168.202.157                  1            476      1.09   
     4 192.168.202.102                  1            422      1.47   
     5 192.168.0.3                      1            108      0.874  
     6 192.168.202.49                   1             90      0.767  
     7 192.168.202.89                   1             78      1.19   
     8 192.168.202.87                   1             48      1.13   
     9 192.168.202.71                   1             43      1.25   
    10 192.168.202.126                  1             40      0.866  

### Шаг 4

Обогащение данных:

#### 10. Определите местоположение (страну, город) и организацию-провайдера для топ-10 доменов. Для этого можно использовать сторонние сервисы, например http://ip-api.com (API-эндпоинт http://ip-api.com/json)

``` r
top_10_d <- dns_d %>% count(query) %>% arrange(desc(n)) %>% head(10) %>% pull(query)

geo_data <- data.frame()
for(domain in top_10_d) {
  url <- paste0("http://ip-api.com/json/", domain)
  response <- GET(url)
  
  if(status_code(response) == 200) {
    info <- content(response)
    geo_data <- rbind(geo_data, data.frame(
      query = domain,
      ip = ifelse(is.null(info$query), NA, info$query),
      country = ifelse(is.null(info$country), NA, info$country),
      city = ifelse(is.null(info$city), NA, info$city),
      isp = ifelse(is.null(info$isp), NA, info$isp)
    ))
  } else {
    geo_data <- rbind(geo_data, data.frame(
      domain = domain, ip = NA, country = NA, city = NA, isp = NA))
  }
  
  Sys.sleep(1)
}

geo_data
```

                                                                         query
    1                                                teredo.ipv6.microsoft.com
    2                                                         tools.google.com
    3                                                            www.apple.com
    4                                                           time.apple.com
    5                                          safebrowsing.clients.google.com
    6  *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00
    7                                                                     WPAD
    8                                              44.206.168.192.in-addr.arpa
    9                                                                 HPE8AA67
    10                                                                  ISATAP
                                                                            ip
    1                                                teredo.ipv6.microsoft.com
    2                                                          142.250.179.174
    3                                                            184.30.157.36
    4                                                            17.253.52.253
    5                                                          142.250.179.206
    6  *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00
    7                                                                     WPAD
    8                                              44.206.168.192.in-addr.arpa
    9                                                                 HPE8AA67
    10                                                                  ISATAP
               country      city                       isp
    1             <NA>      <NA>                      <NA>
    2  The Netherlands Amsterdam                Google LLC
    3  The Netherlands   Haarlem Akamai Technologies, Inc.
    4      Netherlands Amsterdam                Apple Inc.
    5  The Netherlands Amsterdam                Google LLC
    6             <NA>      <NA>                      <NA>
    7             <NA>      <NA>                      <NA>
    8             <NA>      <NA>                      <NA>
    9             <NA>      <NA>                      <NA>
    10            <NA>      <NA>                      <NA>

## Вывод

В ходе выполнения 4 практической работы были озакреплены навыки
использования языка R для исследования метаданных DNS трафика.
