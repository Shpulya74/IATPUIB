# Лабораторная работа №4
Поспелова Ульяна БИСО-03-20

## Цель работы

1.  Закрепить практические навыки использования языка программирования R
    для обработки данных.

2.  Закрепить знания основных функций обработки данных экосистемы
    tidyverse языка R.

3.  Закрепить навыки исследования метаданных DNS трафика.

## Задание

Используя программный пакет dplyr, освоить анализ DNS логов с помощью
языка программирования R.

## Ход работы

### Подготовка данных

1\. Импортируйте данные DNS.

``` r
library(readr)
```

    Warning: пакет 'readr' был собран под R версии 4.3.1

``` r
library(dplyr)
```

    Warning: пакет 'dplyr' был собран под R версии 4.3.1


    Присоединяю пакет: 'dplyr'

    Следующие объекты скрыты от 'package:stats':

        filter, lag

    Следующие объекты скрыты от 'package:base':

        intersect, setdiff, setequal, union

``` r
header <- read.csv("header.csv")
```

``` r
header
```

              Field       Type
    1           ts       time 
    2          uid      string
    3           id      recor 
    4                       d 
    5        proto       proto
    6     trans_id       count
    7        query      string
    8       qclass       count
    9  qclass_name      string
    10       qtype       count
    11  qtype_name      string
    12       rcode       count
    13  rcode_name      string
    14          QR       bool 
    15          AA       bool 
    16       TC RD  bool bool 
    17          RA       bool 
    18           Z       count
    19     answers      vector
    20        TTLs      vector
    21    rejected       bool 
                                                                                           Description
    1                                                                    Timestamp of the DNS request 
    2                                                                     Unique id of the connection 
    3                                                ID record with orig/resp host/port. See conn.log 
    4                                                                                                 
    5                                                        Protocol of DNS transaction – TCP or UDP 
    6                                       16 bit identifier assigned by DNS client; responses match 
    7                                                                Domain name subject of the query 
    8                                                                Value specifying the query class 
    9                                           Descriptive name of the query class (e.g. C_INTERNET) 
    10                                                                Value specifying the query type 
    11                                                     Name of the query type (e.g. A, AAAA, PTR) 
    12                                                        Response code value in the DNS response 
    13                                 Descriptive name of the response code (e.g. NOERROR, NXDOMAIN) 
    14                                        Was this a query or a response? T = response, F = query 
    15                                    Authoritative Answer. T = server is authoritative for query 
    16 Truncation. T = message was truncated Recursion Desired. T = request recursive lookup of query 
    17                                     Recursion Available. T = server supports recursive queries 
    18                                      Reserved field, should be zero in all queries & responses 
    19                                           List of resource descriptions in answer to the query 
    20                                                               Caching intervals of the answers 
    21                                               Whether the DNS query was rejected by the server 

Для чтения логов продублируем первую строку файла, чтобы она не терялась
в названии столбцов.

``` r
dns <- read.csv("dns.log",sep ='\t')
```

2\. Добавьте пропущенные данные о структуре данных (назначении
столбцов).

``` r
names(dns) <- c("ts","uid","id.orig_h","id.orig_p","id.resp_h","id.resp_p","proto","trans_id","query","qclass","qclass_name","qtype","qtype_name","rcode","rcode_name","AA","TC", "RD","RA","Z","answers","TTLs","rejected")
```

3\. Преобразуйте данные в столбцах в нужный формат.

4\. Просмотрите общую структуру данных с помощью функции glimpse().

``` r
dns %>% glimpse()
```

    Rows: 427,935
    Columns: 23
    $ ts          <dbl> 1331901006, 1331901015, 1331901016, 1331901017, 1331901006…
    $ uid         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz7Bs…
    $ id.orig_h   <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", "19…
    $ id.orig_p   <int> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 1…
    $ id.resp_h   <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255", "1…
    $ id.resp_p   <int> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    $ proto       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp", "u…
    $ trans_id    <int> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187, 62…
    $ query       <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\…
    $ qclass      <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1"…
    $ qclass_name <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET", "C…
    $ qtype       <chr> "33", "32", "32", "32", "32", "32", "32", "32", "32", "32"…
    $ qtype_name  <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB…
    $ rcode       <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rcode_name  <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-…
    $ AA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ TC          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ RD          <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRU…
    $ RA          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…
    $ Z           <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0…
    $ answers     <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ TTLs        <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-"…
    $ rejected    <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FA…

### Анализ

5\. Сколько участников информационного обмена в сети Доброй Организации?

``` r
dns %>% group_by(uid) %>% summarize(total = n()) %>% nrow()
```

    [1] 162496

6\. Какое соотношение участников обмена внутри сети и участников
обращений к внешним ресурсам?

``` r
dns %>% filter(qtype_name != "A", qtype_name != "AA", qtype_name != "AAA", qtype_name != "AAAA") %>% group_by(uid) %>% summarize(total = n()) %>% nrow() / dns %>% filter(qtype_name == "A"|qtype_name == "AA"| qtype_name == "AAA" | qtype_name == "AAAA") %>% group_by(uid) %>% summarize(total = n()) %>% nrow()
```

    [1] 0.5084738

7\. Найдите топ-10 участников сети, проявляющих наибольшую сетевую
активность.

``` r
dns %>% select(id.orig_h) %>% group_by(id.orig_h) %>% summarize(total = n()) %>% arrange(desc(total)) %>% head(10)
```

    # A tibble: 10 × 2
       id.orig_h       total
       <chr>           <int>
     1 10.10.117.210   75943
     2 192.168.202.93  26522
     3 192.168.202.103 18121
     4 192.168.202.76  16978
     5 192.168.202.97  16176
     6 192.168.202.141 14967
     7 10.10.117.209   14222
     8 192.168.202.110 13372
     9 192.168.203.63  12148
    10 192.168.202.106 10784

8\. Найдите топ-10 доменов, к которым обращаются пользователи сети и
соответственное количество обращений.

``` r
my_top_10 <- dns %>% select(query, qtype_name) %>% filter(qtype_name == "A"|qtype_name == "AA"| qtype_name == "AAA" | qtype_name == "AAAA") %>% group_by(query) %>% summarize(total = n()) %>% arrange(desc(total)) %>% head(10)
my_top_10
```

    # A tibble: 10 × 2
       query                           total
       <chr>                           <int>
     1 teredo.ipv6.microsoft.com       39273
     2 tools.google.com                14057
     3 www.apple.com                   13390
     4 safebrowsing.clients.google.com 11658
     5 imap.gmail.com                   5543
     6 stats.norton.com                 5537
     7 www.google.com                   5171
     8 ratings-wrs.symantec.com         4464
     9 api.twitter.com                  4348
    10 api.facebook.com                 4137

9\. Опеределите базовые статистические характеристики (функция
summary()) интервала времени между последовательным обращениями к топ-10
доменам.

``` r
summary(diff((dns %>% filter(tolower(query) %in% my_top_10$query) %>% arrange(ts))$ts))
```

        Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
        0.00     0.00     0.00     1.08     0.31 49924.53 

10\. Часто вредоносное программное обеспечение использует DNS канал в
качестве канала управления, периодически отправляя запросы на
подконтрольный злоумышленникам DNS сервер. По периодическим запросам на
один и тот же домен можно выявить скрытый DNS канал. Есть ли такие IP
адреса в исследуемом датасете?

``` r
ip_domain_counts <- dns %>%
  group_by(ip = tolower(id.orig_h), domain = tolower(query)) %>%
  summarise(request_count = n()) %>%
  filter(request_count > 1)
```

    `summarise()` has grouped output by 'ip'. You can override using the `.groups`
    argument.

``` r
unique_ips_with_periodic_requests <- unique(ip_domain_counts$ip)
unique_ips_with_periodic_requests %>% length()
```

    [1] 240

``` r
unique_ips_with_periodic_requests %>% head()
```

    [1] "10.10.10.10"     "10.10.117.209"   "10.10.117.210"   "128.244.37.196" 
    [5] "169.254.109.123" "169.254.228.26" 

### Обогащение данных

11\. Определите местоположение (страну, город) и организацию-провайдера
для топ-10 доменов.

``` r
my_top_10
```

    # A tibble: 10 × 2
       query                           total
       <chr>                           <int>
     1 teredo.ipv6.microsoft.com       39273
     2 tools.google.com                14057
     3 www.apple.com                   13390
     4 safebrowsing.clients.google.com 11658
     5 imap.gmail.com                   5543
     6 stats.norton.com                 5537
     7 www.google.com                   5171
     8 ratings-wrs.symantec.com         4464
     9 api.twitter.com                  4348
    10 api.facebook.com                 4137

1.  teredo.ipv6.microsoft.com: United States, Redmond, Microsoft
    Corporation

2.  tools.google.com: United States, Mountain View, Google

3.  www.apple.com: Germany, Frankfurt, Akamai techonologies

4.  safebrowsing.clients.google.com: United States, Mountain View,
    Google

5.  imap.gmail.com: United States, Iston, Google LLC

6.  stats.norton.com: GreatBritain, Washington, Microsoft Corporation

7.  www.google.com: Unites States, Mountain View, Google

8.  ratings-wrs.symantec.com: United States, Redmond, Microsoft
    Corporation

9.  api.twitter.com: United States, San Francisco, Twitter inc

10. api.facebook.com: Unites States, Menlo Park, Facebook inc

## Оценка результатов

В результате выполнения лабораторной работы были получены ответы на все
поставленные вопросы с помощью языка R и пакета `dplyr`

## Вывод

В ходе выполнения лабораторной работы были импортированы, подготовлены,
проанализированы и обогащены данные DNS трафика.
