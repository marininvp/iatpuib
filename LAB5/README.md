# Основы обработки данных с помощью R

## Практическое задание №5

### Цель работы

#### 1. Закрепить практические навыки использования языка программирования R для обработки данных

#### 2. Закрепить знания основных функций обработки данных экосистемы tidyverse языка R

#### 3. Закрепить навыки исследования метаданных DNS трафика

### Общая ситуация

#### Вы исследуете подозрительную сетевую активность во внутренней сети Доброй Организации. Вам в руки попали метаданные о DNS трафике в исследуемой сети. Исследуйте файлы, восстановите данные, подготовьте их к анализу и дайте обоснованные ответы на поставленные вопросы исследования.

### Задание

### Подготовка данных

### Задание 1

#### 1. Импортируйте данные DNS

    library(dplyr)

    ## 
    ## Присоединяю пакет: 'dplyr'

    ## Следующие объекты скрыты от 'package:stats':
    ## 
    ##     filter, lag

    ## Следующие объекты скрыты от 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    log_data = read.csv("dns.log", header = FALSE,sep = "\t",encoding = "UTF-8")

### Задание 2

#### 2. Добавьте пропущенные данные о структуре данных (назначении столбцов)

    head=read.csv("header.csv", header = TRUE)
    field<-head[,1]
    log_data = read.csv("dns.log", header = FALSE,sep = "\t",encoding = "UTF-8")
    names(log_data)<-field
    log_data%>%glimpse()

    ## Rows: 427,935
    ## Columns: 23
    ## $ `ts `          <dbl> 1331901006, 1331901015, 1331901016, 1331901017, 1331901…
    ## $ `uid `         <chr> "CWGtK431H9XuaTN4fi", "C36a282Jljz7BsbGH", "C36a282Jljz…
    ## $ `orig_ip `     <chr> "192.168.202.100", "192.168.202.76", "192.168.202.76", …
    ## $ `orig_port `   <int> 45658, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137…
    ## $ `resp_ip `     <chr> "192.168.27.203", "192.168.202.255", "192.168.202.255",…
    ## $ `resp_port `   <int> 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, 137, …
    ## $ `proto `       <chr> "udp", "udp", "udp", "udp", "udp", "udp", "udp", "udp",…
    ## $ `trans_id `    <int> 33008, 57402, 57402, 57402, 57398, 57398, 57398, 62187,…
    ## $ `query `       <chr> "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x…
    ## $ `qclass `      <chr> "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", "1", …
    ## $ `qclass_name ` <chr> "C_INTERNET", "C_INTERNET", "C_INTERNET", "C_INTERNET",…
    ## $ `qtype `       <chr> "33", "32", "32", "32", "32", "32", "32", "32", "32", "…
    ## $ `qtype_name `  <chr> "SRV", "NB", "NB", "NB", "NB", "NB", "NB", "NB", "NB", …
    ## $ `rcode `       <chr> "0", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", …
    ## $ `rcode_name `  <chr> "NOERROR", "-", "-", "-", "-", "-", "-", "-", "-", "-",…
    ## $ `QR `          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE,…
    ## $ `AA `          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE,…
    ## $ `TC RD `       <lgl> FALSE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, TRUE, …
    ## $ `RA `          <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE,…
    ## $ `Z `           <int> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1…
    ## $ `answers `     <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", …
    ## $ `TTLs `        <chr> "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", "-", …
    ## $ `rejected `    <lgl> FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE, FALSE,…

### Задание 3

#### 3. Преобразуйте данные в столбцах в нужный формат

### Анализ

### Задание 4

#### Анализ 4. Сколько участников информационного обмена в сети Доброй Организации?

    orig_ip<-unique(log_data$`orig_ip `)
    resp_ip<-unique(log_data$`resp_ip `)
    all_ip<-merge(orig_ip,resp_ip)
    NROW(unique(all_ip$`x`))

    ## [1] 253

### Задание 5

#### 5. Какое соотношение участников обмена внутри сети и участников обращений к внешним ресурсам?

    point51<-unique(log_data$`orig_ip `,sort=TRUE)
    point52<-unique(log_data$`resp_ip `,sort=TRUE)
    collect <- c(point51, point52)
    collect<-collect[!duplicated(collect)]
    lall=length(collect)
    toMatch <- c("(192.168.)([0-9]{1,3}[.])[0-9]{1,3}","(10.0.)([0-9]{1,3}[.])[0-9]{1,3}","(100.64.)([0-9]{1,3}[.])[0-9]{1,3}","(172.16.)([0-9]{1,3}[.])[0-9]{1,3}")
    l1=length(unique(grep(paste(toMatch,collapse="|"), 
                            collect, value=TRUE)))
    l2=lall-l1
    res=l1/l2
    res

    ## [1] 11.94286

### Задание 6

#### 6. Найдите топ-10 участников сети, проявляющих наибольшую сетевую активность.

    point1<-log_data%>%
    count(log_data$`orig_ip `,sort=TRUE)
    colnames(point1) <- c("Person", "count")
    point2<-log_data%>%
    count(log_data$`resp_ip `,sort=TRUE)
    colnames(point2) <- c("Person", "count")
    point_inf<-rbind(point1,point2)
    point_inf<-point_inf %>%
      group_by(Person) %>%
      summarise(count)

    ## `summarise()` has grouped output by 'Person'. You can override using the
    ## `.groups` argument.

    point_inf<-point_inf[order(point_inf$Person, decreasing = TRUE), ]   
    knitr::kable(point_inf, "pipe")

<table>
<thead>
<tr class="header">
<th style="text-align: left;">Person</th>
<th style="text-align: right;">count</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">ff02::fb</td>
<td style="text-align: right;">3298</td>
</tr>
<tr class="even">
<td style="text-align: left;">ff02::1:3</td>
<td style="text-align: right;">14411</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fec0:0:0:ffff::3</td>
<td style="text-align: right;">44</td>
</tr>
<tr class="even">
<td style="text-align: left;">fec0:0:0:ffff::2</td>
<td style="text-align: right;">47</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fec0:0:0:ffff::1</td>
<td style="text-align: right;">47</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::f2de:f1ff:fe9b:ad6a</td>
<td style="text-align: right;">30</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::d840:5635:ef48:b032</td>
<td style="text-align: right;">276</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::d69a:20ff:fef9:b49c</td>
<td style="text-align: right;">141</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::d69a:20ff:fef9:b49c</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::c62c:3ff:fe37:efc</td>
<td style="text-align: right;">126</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::c62c:3ff:fe30:7333</td>
<td style="text-align: right;">186</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::c62c:3ff:fe30:7333</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::bc5c:15c1:ec81:1e08</td>
<td style="text-align: right;">8669</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::ba8d:12ff:fe53:a8d8</td>
<td style="text-align: right;">252</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::ba8d:12ff:fe53:a8d8</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::ba8d:12ff:fe12:3f90</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::ba8d:12ff:fe12:3f90</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::a800:4ff:fe00:a04</td>
<td style="text-align: right;">429</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::9c8a:5786:bbb0:3db8</td>
<td style="text-align: right;">240</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::88f4:9f4d:feec:8072</td>
<td style="text-align: right;">48</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::65ca:c6cd:7ae0:ac8c</td>
<td style="text-align: right;">528</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::62fb:42ff:feef:5440</td>
<td style="text-align: right;">73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::62c5:47ff:fe93:381e</td>
<td style="text-align: right;">5</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::4c9b:aad8:8a6a:7bb0</td>
<td style="text-align: right;">268</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::4c3a:e571:4cfc:b70c</td>
<td style="text-align: right;">412</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::423c:fcff:fe06:98a4</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::4172:3555:4717:3e0c</td>
<td style="text-align: right;">873</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::3ed0:f8ff:fe34:4765</td>
<td style="text-align: right;">11</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::3e07:54ff:fe41:3ed3</td>
<td style="text-align: right;">82</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::3e07:54ff:fe1c:a665</td>
<td style="text-align: right;">724</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::223:dfff:fe97:4e12</td>
<td style="text-align: right;">74</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::216:d3ff:fe4b:70d</td>
<td style="text-align: right;">11</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::20c:29ff:fe93:571e</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::20c:29ff:fe41:4be7</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">fe80::11a:f507:d853:a03d</td>
<td style="text-align: right;">3097</td>
</tr>
<tr class="even">
<td style="text-align: left;">8.8.8.8</td>
<td style="text-align: right;">506</td>
</tr>
<tr class="odd">
<td style="text-align: left;">8.8.4.4</td>
<td style="text-align: right;">304</td>
</tr>
<tr class="even">
<td style="text-align: left;">8.26.56.26</td>
<td style="text-align: right;">5974</td>
</tr>
<tr class="odd">
<td style="text-align: left;">72.14.204.97</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">72.14.204.120</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">72.14.204.100</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">68.87.75.198</td>
<td style="text-align: right;">4171</td>
</tr>
<tr class="odd">
<td style="text-align: left;">68.87.64.150</td>
<td style="text-align: right;">4838</td>
</tr>
<tr class="even">
<td style="text-align: left;">64.134.255.2</td>
<td style="text-align: right;">477</td>
</tr>
<tr class="odd">
<td style="text-align: left;">64.134.255.10</td>
<td style="text-align: right;">483</td>
</tr>
<tr class="even">
<td style="text-align: left;">64.102.255.44</td>
<td style="text-align: right;">65</td>
</tr>
<tr class="odd">
<td style="text-align: left;">255.255.255.255</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">224.0.0.251</td>
<td style="text-align: right;">24</td>
</tr>
<tr class="odd">
<td style="text-align: left;">208.67.222.222</td>
<td style="text-align: right;">65</td>
</tr>
<tr class="even">
<td style="text-align: left;">208.67.220.220</td>
<td style="text-align: right;">65</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:855:5894:e447:d5c5:3d0c</td>
<td style="text-align: right;">159</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:855::25</td>
<td style="text-align: right;">148</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:755:f80b:a003:fa7a:9852</td>
<td style="text-align: right;">152</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:755::25</td>
<td style="text-align: right;">146</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:655:391c:d28f:fa5d:495</td>
<td style="text-align: right;">157</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:655:20c:29ff:fe47:e982</td>
<td style="text-align: right;">102</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:655:20c:29ff:fe28:6955</td>
<td style="text-align: right;">99</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:655:20c:29ff:fe1f:96f2</td>
<td style="text-align: right;">109</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:655:190:45fb:a92c:ba4f</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:655::25</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:555:f8d9:a1ea:b86e:10ad</td>
<td style="text-align: right;">836</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:555:c814:8f50:bbf5:ea5b</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:555::25</td>
<td style="text-align: right;">841</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:555::103</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:204:a800:4ff:fe00:a04</td>
<td style="text-align: right;">763</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:203:226:18ff:fef9:be98</td>
<td style="text-align: right;">9</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:f2de:f1ff:fe9b:ad6a</td>
<td style="text-align: right;">214</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:d557:eac5:3728:41ee</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:d4bc:e39f:84ad:5001</td>
<td style="text-align: right;">63</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:bc5c:15c1:ec81:1e08</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:b9ac:6976:ae1c:e10b</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:a800:4ff:fe00:a04</td>
<td style="text-align: right;">73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:a1dd:6355:28ae:da1e</td>
<td style="text-align: right;">9</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:9c6b:be4e:1289:2663</td>
<td style="text-align: right;">15</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:71f3:e219:3f6a:4311</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:6128:bec8:28c6:8c7b</td>
<td style="text-align: right;">24</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:4c3a:e571:4cfc:b70c</td>
<td style="text-align: right;">5</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:4172:3555:4717:3e0c</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:216:d3ff:fe4b:70d</td>
<td style="text-align: right;">19</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:20c:29ff:fe93:571e</td>
<td style="text-align: right;">1392</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:20c:29ff:fe78:1023</td>
<td style="text-align: right;">25</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:20c:29ff:fe41:4be7</td>
<td style="text-align: right;">1379</td>
</tr>
<tr class="odd">
<td style="text-align: left;">198.153.194.1</td>
<td style="text-align: right;">10</td>
</tr>
<tr class="even">
<td style="text-align: left;">198.153.192.1</td>
<td style="text-align: right;">10</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.95.255</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.95.166</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.94.36</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.51.38</td>
<td style="text-align: right;">74</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.51.255</td>
<td style="text-align: right;">74</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.99</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.98</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.97</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.96</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.95</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.94</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.93</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.92</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.91</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.90</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.9</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.89</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.88</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.87</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.86</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.85</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.84</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.83</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.82</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.81</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.80</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.8</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.79</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.78</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.77</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.76</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.75</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.74</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.73</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.72</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.71</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.70</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.7</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.69</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.68</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.67</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.66</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.65</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.64</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.63</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.62</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.61</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.60</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.6</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.59</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.58</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.57</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.56</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.55</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.54</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.53</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.52</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.51</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.50</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.5</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.49</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.48</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.47</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.46</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.45</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.44</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.43</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.42</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.41</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.40</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.4</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.39</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.38</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.37</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.36</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.35</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.34</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.33</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.32</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.31</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.30</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.3</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.29</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.28</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.27</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.26</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.254</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.254</td>
<td style="text-align: right;">47</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.253</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.253</td>
<td style="text-align: right;">36</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.252</td>
<td style="text-align: right;">24</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.251</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.250</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.25</td>
<td style="text-align: right;">1599</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.25</td>
<td style="text-align: right;">205</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.249</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.248</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.247</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.246</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.245</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.244</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.243</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.242</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.241</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.240</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.24</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.239</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.238</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.237</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.236</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.235</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.234</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.233</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.232</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.231</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.230</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.23</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.229</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.228</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.227</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.226</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.225</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.224</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.223</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.222</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.221</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.220</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.22</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.219</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.218</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.217</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.216</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.215</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.214</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.213</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.212</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.211</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.210</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.21</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.209</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.208</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.207</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.206</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.205</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.204</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.203</td>
<td style="text-align: right;">64</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.202</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.202</td>
<td style="text-align: right;">80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.201</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.200</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.20</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.2</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.199</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.198</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.197</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.196</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.195</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.194</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.193</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.192</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.191</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.190</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.19</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.189</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.188</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.187</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.186</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.185</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.184</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.183</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.182</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.181</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.180</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.18</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.179</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.178</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.177</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.176</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.175</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.174</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.173</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.172</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.171</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.170</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.17</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.169</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.168</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.167</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.166</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.165</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.164</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.163</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.162</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.161</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.160</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.16</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.159</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.158</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.157</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.156</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.155</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.154</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.153</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.152</td>
<td style="text-align: right;">34</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.152</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.151</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.150</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.15</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.149</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.148</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.147</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.146</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.145</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.144</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.143</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.142</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.141</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.140</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.14</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.139</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.138</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.137</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.136</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.135</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.134</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.133</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.132</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.131</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.130</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.13</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.129</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.128</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.127</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.126</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.125</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.124</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.123</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.122</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.121</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.120</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.12</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.119</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.118</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.117</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.116</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.115</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.114</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.113</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.112</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.111</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.110</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.11</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.109</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.108</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.107</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.106</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.105</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.104</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.103</td>
<td style="text-align: right;">23</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.103</td>
<td style="text-align: right;">33</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.102</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.102</td>
<td style="text-align: right;">97</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.101</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.101</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.100</td>
<td style="text-align: right;">21</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.10</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.1</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.1</td>
<td style="text-align: right;">42</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.0</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.99</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.98</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.97</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.96</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.95</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.94</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.93</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.92</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.91</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.90</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.9</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.89</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.88</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.87</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.86</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.85</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.84</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.83</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.82</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.81</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.80</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.8</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.79</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.78</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.77</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.76</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.75</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.74</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.73</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.72</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.71</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.70</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.7</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.69</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.68</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.67</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.66</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.65</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.64</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.63</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.62</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.61</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.60</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.6</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.59</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.58</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.57</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.56</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.55</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.54</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.53</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.52</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.51</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.50</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.5</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.49</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.48</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.47</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.46</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.45</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.44</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.43</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.42</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.41</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.40</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.4</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.39</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.38</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.37</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.36</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.35</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.34</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.33</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.32</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.31</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.30</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.3</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.29</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.28</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.27</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.26</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.254</td>
<td style="text-align: right;">20</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.254</td>
<td style="text-align: right;">59</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.253</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.253</td>
<td style="text-align: right;">40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.252</td>
<td style="text-align: right;">17</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.251</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.250</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.25</td>
<td style="text-align: right;">534</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.25</td>
<td style="text-align: right;">181</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.249</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.248</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.247</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.246</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.245</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.244</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.243</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.242</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.241</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.240</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.24</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.239</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.238</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.237</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.236</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.235</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.234</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.233</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.232</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.231</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.230</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.23</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.229</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.228</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.227</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.226</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.225</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.224</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.223</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.222</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.221</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.220</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.22</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.219</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.218</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.217</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.216</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.215</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.214</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.213</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.212</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.211</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.210</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.21</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.209</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.208</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.207</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.206</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.205</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.204</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.203</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.203</td>
<td style="text-align: right;">95</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.202</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.202</td>
<td style="text-align: right;">76</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.201</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.200</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.20</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.2</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.199</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.198</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.197</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.196</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.195</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.194</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.193</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.192</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.191</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.190</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.19</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.189</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.188</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.187</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.186</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.185</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.184</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.183</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.182</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.181</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.180</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.18</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.179</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.178</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.177</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.176</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.175</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.174</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.173</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.172</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.171</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.170</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.17</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.169</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.168</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.167</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.166</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.165</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.164</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.163</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.162</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.161</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.160</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.16</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.159</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.158</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.157</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.156</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.155</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.154</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.153</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.152</td>
<td style="text-align: right;">46</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.152</td>
<td style="text-align: right;">45</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.151</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.150</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.15</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.149</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.148</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.147</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.146</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.145</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.144</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.143</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.142</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.141</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.140</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.14</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.139</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.138</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.137</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.136</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.135</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.134</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.133</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.132</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.131</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.130</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.13</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.129</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.128</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.127</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.126</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.125</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.124</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.123</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.122</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.121</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.120</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.12</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.119</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.118</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.117</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.116</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.115</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.114</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.113</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.112</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.111</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.110</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.11</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.109</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.108</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.107</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.106</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.105</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.104</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.103</td>
<td style="text-align: right;">30</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.103</td>
<td style="text-align: right;">51</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.102</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.102</td>
<td style="text-align: right;">118</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.101</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.101</td>
<td style="text-align: right;">44</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.100</td>
<td style="text-align: right;">86</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.100</td>
<td style="text-align: right;">52</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.10</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.1</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.1</td>
<td style="text-align: right;">66</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.0</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.84</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.8</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.6</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.4</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.38</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.37</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.36</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.34</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.32</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.30</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.28</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.26</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.254</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.254</td>
<td style="text-align: right;">43</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.253</td>
<td style="text-align: right;">8</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.252</td>
<td style="text-align: right;">21</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.25</td>
<td style="text-align: right;">1024</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.25</td>
<td style="text-align: right;">112</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.24</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.22</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.203</td>
<td style="text-align: right;">64</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.202</td>
<td style="text-align: right;">78</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.20</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.2</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.18</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.178</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.16</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.152</td>
<td style="text-align: right;">96</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.152</td>
<td style="text-align: right;">28</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.14</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.131</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.12</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.103</td>
<td style="text-align: right;">83</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.103</td>
<td style="text-align: right;">17</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.102</td>
<td style="text-align: right;">65</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.101</td>
<td style="text-align: right;">9</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.100</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.10</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.1</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.1</td>
<td style="text-align: right;">39</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.0</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.58</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.254</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.254</td>
<td style="text-align: right;">44</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.253</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.253</td>
<td style="text-align: right;">23</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.252</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.25</td>
<td style="text-align: right;">3655</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.25</td>
<td style="text-align: right;">242</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.246</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.203</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.203</td>
<td style="text-align: right;">54</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.202</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.202</td>
<td style="text-align: right;">42</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.199</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.152</td>
<td style="text-align: right;">300</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.152</td>
<td style="text-align: right;">25</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.11</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.105</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.103</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.103</td>
<td style="text-align: right;">69</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.102</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.102</td>
<td style="text-align: right;">47</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.101</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.101</td>
<td style="text-align: right;">25</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.100</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.100</td>
<td style="text-align: right;">61</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.1</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.1</td>
<td style="text-align: right;">36</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.99</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.98</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.97</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.96</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.95</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.94</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.93</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.92</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.91</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.90</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.9</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.89</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.88</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.87</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.86</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.85</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.84</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.83</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.82</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.81</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.80</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.8</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.79</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.78</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.77</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.76</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.75</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.74</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.73</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.72</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.71</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.70</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.7</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.69</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.68</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.67</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.66</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.65</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.64</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.63</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.62</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.61</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.60</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.6</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.59</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.58</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.57</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.56</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.55</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.54</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.53</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.52</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.51</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.50</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.5</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.49</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.48</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.47</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.46</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.45</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.44</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.43</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.42</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.41</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.40</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.4</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.39</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.38</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.37</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.36</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.35</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.34</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.33</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.32</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.31</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.30</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.3</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.29</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.28</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.27</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.26</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.254</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.254</td>
<td style="text-align: right;">42</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.253</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.253</td>
<td style="text-align: right;">23</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.252</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.252</td>
<td style="text-align: right;">25</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.251</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.250</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.25</td>
<td style="text-align: right;">1563</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.25</td>
<td style="text-align: right;">70</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.249</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.248</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.247</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.246</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.245</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.244</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.243</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.242</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.241</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.240</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.24</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.239</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.238</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.237</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.236</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.235</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.234</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.233</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.232</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.231</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.230</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.23</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.229</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.228</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.227</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.226</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.225</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.224</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.223</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.222</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.221</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.220</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.22</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.219</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.218</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.217</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.216</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.215</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.214</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.213</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.212</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.211</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.210</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.21</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.209</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.208</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.207</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.206</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.205</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.204</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.203</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.203</td>
<td style="text-align: right;">51</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.202</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.202</td>
<td style="text-align: right;">68</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.201</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.200</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.20</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.2</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.199</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.198</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.197</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.196</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.195</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.194</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.193</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.192</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.191</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.190</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.19</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.189</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.188</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.187</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.186</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.185</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.184</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.183</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.182</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.181</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.180</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.18</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.179</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.178</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.177</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.176</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.175</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.174</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.173</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.172</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.171</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.170</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.17</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.169</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.168</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.167</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.166</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.165</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.164</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.163</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.162</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.161</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.160</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.16</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.159</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.158</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.157</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.156</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.155</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.154</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.153</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.152</td>
<td style="text-align: right;">9</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.152</td>
<td style="text-align: right;">29</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.151</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.150</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.15</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.149</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.148</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.147</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.146</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.145</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.144</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.143</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.142</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.141</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.140</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.14</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.139</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.138</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.137</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.136</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.135</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.134</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.133</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.132</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.131</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.130</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.13</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.129</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.128</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.127</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.126</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.125</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.124</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.123</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.122</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.121</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.120</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.12</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.119</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.118</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.117</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.116</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.115</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.114</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.113</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.112</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.111</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.110</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.11</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.109</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.108</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.107</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.106</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.105</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.104</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.103</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.103</td>
<td style="text-align: right;">38</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.102</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.102</td>
<td style="text-align: right;">50</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.101</td>
<td style="text-align: right;">13</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.100</td>
<td style="text-align: right;">25</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.10</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.1</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.1</td>
<td style="text-align: right;">40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.0</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.6</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.53</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.254</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.254</td>
<td style="text-align: right;">59</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.253</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.253</td>
<td style="text-align: right;">24</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.252</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.252</td>
<td style="text-align: right;">17</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.25</td>
<td style="text-align: right;">1303</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.25</td>
<td style="text-align: right;">149</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.241</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.203</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.203</td>
<td style="text-align: right;">64</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.202</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.202</td>
<td style="text-align: right;">67</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.194</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.152</td>
<td style="text-align: right;">14</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.152</td>
<td style="text-align: right;">43</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.147</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.103</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.103</td>
<td style="text-align: right;">55</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.102</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.102</td>
<td style="text-align: right;">67</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.101</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.101</td>
<td style="text-align: right;">39</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.100</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.100</td>
<td style="text-align: right;">60</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.1</td>
<td style="text-align: right;">13</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.1</td>
<td style="text-align: right;">52</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.254</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.254</td>
<td style="text-align: right;">21</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.252</td>
<td style="text-align: right;">9530</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.252</td>
<td style="text-align: right;">71</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.251</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.251</td>
<td style="text-align: right;">98</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.156</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.156</td>
<td style="text-align: right;">63</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.153</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.153</td>
<td style="text-align: right;">29</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.101</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.101</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.1</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.1</td>
<td style="text-align: right;">15</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.254</td>
<td style="text-align: right;">27</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.254</td>
<td style="text-align: right;">73</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.253</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.253</td>
<td style="text-align: right;">40</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.252</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.252</td>
<td style="text-align: right;">35</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.25</td>
<td style="text-align: right;">985</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.25</td>
<td style="text-align: right;">193</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.215</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.203</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.203</td>
<td style="text-align: right;">117</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.202</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.202</td>
<td style="text-align: right;">71</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.152</td>
<td style="text-align: right;">80</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.152</td>
<td style="text-align: right;">46</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.103</td>
<td style="text-align: right;">14</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.103</td>
<td style="text-align: right;">49</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.102</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.102</td>
<td style="text-align: right;">146</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.101</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.101</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.100</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.100</td>
<td style="text-align: right;">26</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.1</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.1</td>
<td style="text-align: right;">64</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.99</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.98</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.97</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.96</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.95</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.94</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.93</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.92</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.91</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.90</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.9</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.89</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.88</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.87</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.86</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.85</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.84</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.83</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.82</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.81</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.80</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.8</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.79</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.78</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.77</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.76</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.75</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.74</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.73</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.72</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.71</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.70</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.7</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.69</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.68</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.67</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.66</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.65</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.64</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.63</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.62</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.61</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.60</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.6</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.59</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.58</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.57</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.56</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.55</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.54</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.53</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.52</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.51</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.50</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.5</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.49</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.48</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.47</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.46</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.45</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.44</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.43</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.42</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.41</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.40</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.4</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.39</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.38</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.37</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.36</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.35</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.34</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.33</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.32</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.31</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.30</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.3</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.29</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.28</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.27</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.26</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.254</td>
<td style="text-align: right;">20</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.254</td>
<td style="text-align: right;">72</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.253</td>
<td style="text-align: right;">5</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.253</td>
<td style="text-align: right;">42</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.252</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.252</td>
<td style="text-align: right;">22</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.251</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.250</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.25</td>
<td style="text-align: right;">2509</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.25</td>
<td style="text-align: right;">226</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.249</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.248</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.247</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.246</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.245</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.244</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.243</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.242</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.241</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.240</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.24</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.239</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.238</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.237</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.236</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.235</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.234</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.233</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.232</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.231</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.230</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.23</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.229</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.228</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.227</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.226</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.225</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.224</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.223</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.222</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.221</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.220</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.22</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.219</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.218</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.217</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.216</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.215</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.214</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.213</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.212</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.211</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.210</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.21</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.209</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.208</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.207</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.206</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.205</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.204</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.203</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.203</td>
<td style="text-align: right;">105</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.202</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.202</td>
<td style="text-align: right;">90</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.201</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.200</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.20</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.2</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.199</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.198</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.197</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.196</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.195</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.194</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.193</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.192</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.191</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.190</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.19</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.189</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.188</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.187</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.186</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.185</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.184</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.183</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.182</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.181</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.180</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.18</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.179</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.178</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.177</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.176</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.175</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.174</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.173</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.172</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.171</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.170</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.17</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.169</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.168</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.167</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.166</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.165</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.164</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.163</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.162</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.161</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.160</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.16</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.159</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.158</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.157</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.156</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.155</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.154</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.153</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.152</td>
<td style="text-align: right;">773</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.152</td>
<td style="text-align: right;">55</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.151</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.150</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.15</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.149</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.148</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.147</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.146</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.145</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.144</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.143</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.142</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.141</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.140</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.14</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.139</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.138</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.137</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.136</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.135</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.134</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.133</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.132</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.131</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.130</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.13</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.129</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.128</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.127</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.126</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.125</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.124</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.123</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.122</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.121</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.120</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.12</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.119</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.118</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.117</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.116</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.115</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.114</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.113</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.112</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.111</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.110</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.11</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.109</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.108</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.107</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.106</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.105</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.104</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.103</td>
<td style="text-align: right;">2159</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.103</td>
<td style="text-align: right;">66</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.102</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.102</td>
<td style="text-align: right;">142</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.101</td>
<td style="text-align: right;">5</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.100</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.100</td>
<td style="text-align: right;">55</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.10</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.1</td>
<td style="text-align: right;">20</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.1</td>
<td style="text-align: right;">59</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.0</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.208.18</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.207.58</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.207.4</td>
<td style="text-align: right;">85</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.207.4</td>
<td style="text-align: right;">266542</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.206.44</td>
<td style="text-align: right;">1852</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.205.253</td>
<td style="text-align: right;">422</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.205.1</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.204.72</td>
<td style="text-align: right;">68</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.204.71</td>
<td style="text-align: right;">50</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.204.70</td>
<td style="text-align: right;">4899</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.204.69</td>
<td style="text-align: right;">11</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.204.60</td>
<td style="text-align: right;">3452</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.204.59</td>
<td style="text-align: right;">1583</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.204.45</td>
<td style="text-align: right;">1727</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.204.45</td>
<td style="text-align: right;">51</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.204.255</td>
<td style="text-align: right;">1434</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.203.66</td>
<td style="text-align: right;">120</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.203.64</td>
<td style="text-align: right;">2413</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.203.64</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.203.63</td>
<td style="text-align: right;">12148</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.203.62</td>
<td style="text-align: right;">742</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.203.61</td>
<td style="text-align: right;">589</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.203.45</td>
<td style="text-align: right;">1478</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.203.45</td>
<td style="text-align: right;">108</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.203.255</td>
<td style="text-align: right;">74</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.98</td>
<td style="text-align: right;">440</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.98</td>
<td style="text-align: right;">15</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.97</td>
<td style="text-align: right;">16176</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.96</td>
<td style="text-align: right;">133</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.96</td>
<td style="text-align: right;">24</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.95</td>
<td style="text-align: right;">682</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.94</td>
<td style="text-align: right;">3040</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.94</td>
<td style="text-align: right;">21</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.93</td>
<td style="text-align: right;">26522</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.92</td>
<td style="text-align: right;">213</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.91</td>
<td style="text-align: right;">1301</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.90</td>
<td style="text-align: right;">1584</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.89</td>
<td style="text-align: right;">2088</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.88</td>
<td style="text-align: right;">4169</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.87</td>
<td style="text-align: right;">5122</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.86</td>
<td style="text-align: right;">5</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.85</td>
<td style="text-align: right;">5227</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.84</td>
<td style="text-align: right;">8244</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.83</td>
<td style="text-align: right;">10685</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.81</td>
<td style="text-align: right;">65</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.80</td>
<td style="text-align: right;">4961</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.80</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.79</td>
<td style="text-align: right;">10580</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.79</td>
<td style="text-align: right;">908</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.78</td>
<td style="text-align: right;">965</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.78</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.77</td>
<td style="text-align: right;">3834</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.76</td>
<td style="text-align: right;">16978</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.75</td>
<td style="text-align: right;">6840</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.75</td>
<td style="text-align: right;">5</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.74</td>
<td style="text-align: right;">18</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.73</td>
<td style="text-align: right;">282</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.71</td>
<td style="text-align: right;">7375</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.68</td>
<td style="text-align: right;">43</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.68</td>
<td style="text-align: right;">15</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.67</td>
<td style="text-align: right;">5</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.65</td>
<td style="text-align: right;">1782</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.64</td>
<td style="text-align: right;">722</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.64</td>
<td style="text-align: right;">33</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.63</td>
<td style="text-align: right;">790</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.63</td>
<td style="text-align: right;">21</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.62</td>
<td style="text-align: right;">1441</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.49</td>
<td style="text-align: right;">3402</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.4</td>
<td style="text-align: right;">34</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.4</td>
<td style="text-align: right;">303</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.33</td>
<td style="text-align: right;">172</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.255</td>
<td style="text-align: right;">68720</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.253</td>
<td style="text-align: right;">68</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.240</td>
<td style="text-align: right;">1217</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.222</td>
<td style="text-align: right;">144</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.157</td>
<td style="text-align: right;">1491</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.156</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.155</td>
<td style="text-align: right;">180</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.154</td>
<td style="text-align: right;">63</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.153</td>
<td style="text-align: right;">100</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.152</td>
<td style="text-align: right;">691</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.152</td>
<td style="text-align: right;">51</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.151</td>
<td style="text-align: right;">96</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.150</td>
<td style="text-align: right;">951</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.149</td>
<td style="text-align: right;">232</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.148</td>
<td style="text-align: right;">8</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.147</td>
<td style="text-align: right;">72</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.146</td>
<td style="text-align: right;">28</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.145</td>
<td style="text-align: right;">712</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.144</td>
<td style="text-align: right;">257</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.144</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.143</td>
<td style="text-align: right;">257</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.141</td>
<td style="text-align: right;">14967</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.141</td>
<td style="text-align: right;">9</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.140</td>
<td style="text-align: right;">5021</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.140</td>
<td style="text-align: right;">863</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.139</td>
<td style="text-align: right;">785</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.138</td>
<td style="text-align: right;">5022</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.138</td>
<td style="text-align: right;">496</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.137</td>
<td style="text-align: right;">1749</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.136</td>
<td style="text-align: right;">452</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.136</td>
<td style="text-align: right;">69</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.135</td>
<td style="text-align: right;">239</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.135</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.134</td>
<td style="text-align: right;">104</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.133</td>
<td style="text-align: right;">829</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.132</td>
<td style="text-align: right;">96</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.131</td>
<td style="text-align: right;">174</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.130</td>
<td style="text-align: right;">249</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.129</td>
<td style="text-align: right;">290</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.128</td>
<td style="text-align: right;">94</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.126</td>
<td style="text-align: right;">152</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.125</td>
<td style="text-align: right;">186</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.124</td>
<td style="text-align: right;">16</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.123</td>
<td style="text-align: right;">1511</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.122</td>
<td style="text-align: right;">254</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.122</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.121</td>
<td style="text-align: right;">983</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.120</td>
<td style="text-align: right;">993</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.119</td>
<td style="text-align: right;">39</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.119</td>
<td style="text-align: right;">36</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.118</td>
<td style="text-align: right;">648</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.118</td>
<td style="text-align: right;">249</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.117</td>
<td style="text-align: right;">184</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.116</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.115</td>
<td style="text-align: right;">5039</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.115</td>
<td style="text-align: right;">12</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.113</td>
<td style="text-align: right;">3863</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.112</td>
<td style="text-align: right;">3595</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.112</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.110</td>
<td style="text-align: right;">13372</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.110</td>
<td style="text-align: right;">1121</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.109</td>
<td style="text-align: right;">315</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.108</td>
<td style="text-align: right;">2259</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.108</td>
<td style="text-align: right;">48</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.107</td>
<td style="text-align: right;">141</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.106</td>
<td style="text-align: right;">10784</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.103</td>
<td style="text-align: right;">18121</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.102</td>
<td style="text-align: right;">7565</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.102</td>
<td style="text-align: right;">199</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.101</td>
<td style="text-align: right;">947</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.101</td>
<td style="text-align: right;">93</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.100</td>
<td style="text-align: right;">1367</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.202.100</td>
<td style="text-align: right;">30</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.1</td>
<td style="text-align: right;">122</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.2.100</td>
<td style="text-align: right;">9</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.169.2</td>
<td style="text-align: right;">8</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.100.255</td>
<td style="text-align: right;">486</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.100.130</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.1.255</td>
<td style="text-align: right;">712</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.1.1</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.0.3</td>
<td style="text-align: right;">266</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.0.255</td>
<td style="text-align: right;">270</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.0.199</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.0.1</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">184.49.188.129</td>
<td style="text-align: right;">486</td>
</tr>
<tr class="even">
<td style="text-align: left;">173.194.73.106</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.19.1.100</td>
<td style="text-align: right;">25481</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.8.25</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.16.8.102</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.7.100</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.16.6.57</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.6.25</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.16.5.25</td>
<td style="text-align: right;">33</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.48.159</td>
<td style="text-align: right;">2</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.16.42.42</td>
<td style="text-align: right;">4962</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.42.255</td>
<td style="text-align: right;">4962</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.16.4.25</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.4.102</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.16.3.100</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.2.25</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.16.10.2</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.10.130</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">172.16.1.254</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">169.254.255.255</td>
<td style="text-align: right;">29</td>
</tr>
<tr class="odd">
<td style="text-align: left;">169.254.228.26</td>
<td style="text-align: right;">24</td>
</tr>
<tr class="even">
<td style="text-align: left;">169.254.109.123</td>
<td style="text-align: right;">4</td>
</tr>
<tr class="odd">
<td style="text-align: left;">156.154.70.22</td>
<td style="text-align: right;">5821</td>
</tr>
<tr class="even">
<td style="text-align: left;">128.244.37.255</td>
<td style="text-align: right;">31</td>
</tr>
<tr class="odd">
<td style="text-align: left;">128.244.37.196</td>
<td style="text-align: right;">31</td>
</tr>
<tr class="even">
<td style="text-align: left;">128.244.177.34</td>
<td style="text-align: right;">7</td>
</tr>
<tr class="odd">
<td style="text-align: left;">128.244.177.18</td>
<td style="text-align: right;">6</td>
</tr>
<tr class="even">
<td style="text-align: left;">128.244.172.252</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">110.209.6.25</td>
<td style="text-align: right;">9</td>
</tr>
<tr class="even">
<td style="text-align: left;">10.7.137.108</td>
<td style="text-align: right;">243</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10.7.136.63</td>
<td style="text-align: right;">237</td>
</tr>
<tr class="even">
<td style="text-align: left;">10.7.136.159</td>
<td style="text-align: right;">246</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10.7.136.109</td>
<td style="text-align: right;">240</td>
</tr>
<tr class="even">
<td style="text-align: left;">10.255.255.255</td>
<td style="text-align: right;">69</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10.133.20.11</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">10.105.2.72</td>
<td style="text-align: right;">3</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10.10.7.253</td>
<td style="text-align: right;">1</td>
</tr>
<tr class="even">
<td style="text-align: left;">10.10.117.210</td>
<td style="text-align: right;">75943</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10.10.117.209</td>
<td style="text-align: right;">14222</td>
</tr>
<tr class="even">
<td style="text-align: left;">10.10.10.10</td>
<td style="text-align: right;">69</td>
</tr>
<tr class="odd">
<td style="text-align: left;">10.10.0.7</td>
<td style="text-align: right;">9</td>
</tr>
</tbody>
</table>

### Задание 7

#### 7. Найдите топ-10 доменов, к которым обращаются пользователи сети и соответственное количество обращений

    point_7<-log_data%>%
    count(log_data$`query `,sort=TRUE)
    point_7_head<-point_7%>%
    head(10)
    point_7_head

    ##                                                          log_data$`query `
    ## 1                                                teredo.ipv6.microsoft.com
    ## 2                                                         tools.google.com
    ## 3                                                            www.apple.com
    ## 4                                                           time.apple.com
    ## 5                                          safebrowsing.clients.google.com
    ## 6  *\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00
    ## 7                                                                     WPAD
    ## 8                                              44.206.168.192.in-addr.arpa
    ## 9                                                                 HPE8AA67
    ## 10                                                                  ISATAP
    ##        n
    ## 1  39273
    ## 2  14057
    ## 3  13390
    ## 4  13109
    ## 5  11658
    ## 6  10401
    ## 7   9134
    ## 8   7248
    ## 9   6929
    ## 10  6569

### Задание 8

#### 8. Опеределите базовые статистические характеристики (функция summary()) интервала времени между последовательным обращениями к топ-10 доменам.

    point_7_head%>%summary()

    ##  log_data$`query `        n        
    ##  Length:10          Min.   : 6569  
    ##  Class :character   1st Qu.: 7720  
    ##  Mode  :character   Median :11030  
    ##                     Mean   :13177  
    ##                     3rd Qu.:13320  
    ##                     Max.   :39273

### Задание 9

#### 9. Часто вредоносное программное обеспечение использует DNS канал в качестве канала управления, периодически отправляя запросы на подконтрольный злоумышленникам DNS сервер. По периодическим запросам на один и тот же домен можно выявить скрытый DNS канал. Есть ли такие IP адреса в исследуемом датасете?

    point_9s<-log_data[,c("orig_ip ","query ")]
    point_8<-point_9s
    point_8%>%group_by(point_8$`query `)

    ## # A tibble: 427,935 × 3
    ## # Groups:   point_8$`query ` [5,178]
    ##    `orig_ip `      `query `                                              point…¹
    ##    <chr>           <chr>                                                 <chr>  
    ##  1 192.168.202.100 "*\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00… "*\\x0…
    ##  2 192.168.202.76  "HPE8AA67"                                            "HPE8A…
    ##  3 192.168.202.76  "HPE8AA67"                                            "HPE8A…
    ##  4 192.168.202.76  "HPE8AA67"                                            "HPE8A…
    ##  5 192.168.202.76  "WPAD"                                                "WPAD" 
    ##  6 192.168.202.76  "WPAD"                                                "WPAD" 
    ##  7 192.168.202.76  "WPAD"                                                "WPAD" 
    ##  8 192.168.202.89  "EWREP1"                                              "EWREP…
    ##  9 192.168.202.89  "EWREP1"                                              "EWREP…
    ## 10 192.168.202.89  "EWREP1"                                              "EWREP…
    ## # … with 427,925 more rows, and abbreviated variable name ¹​`point_8$\`query \``

    point_8<-point_8%>%group_by(point_8$orig_ip ) %>%count(point_8$`query `,sort=TRUE)
    point_8<-point_8%>%count(point_8$`point_8$orig_ip`,)
    point_8<-point_8[which(point_8$n==1),]
    knitr::kable(point_8[,c('point_8$orig_ip')], "pipe")

<table>
<thead>
<tr class="header">
<th style="text-align: left;">point_8$orig_ip</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">128.244.172.252</td>
</tr>
<tr class="even">
<td style="text-align: left;">172.16.10.130</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.100.130</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.202.116</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.207.58</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.208.18</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.100</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.102</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.202</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.203</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.252</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.21.253</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.21.254</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.100</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.101</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.102</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.202</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.203</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.22.252</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.22.253</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.101</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.153</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.156</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.229.251</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.229.254</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.100</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.101</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.102</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.103</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.202</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.203</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.23.252</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.23.253</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.102</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.103</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.202</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.203</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.252</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.24.253</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.24.254</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.100</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.101</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.102</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.103</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.152</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.202</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.25.203</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.25.253</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.103</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.26.152</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.26.254</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.1</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.100</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.101</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.102</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.202</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.27.203</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.27.253</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.1</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.101</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.102</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.103</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.152</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.202</td>
</tr>
<tr class="even">
<td style="text-align: left;">192.168.28.253</td>
</tr>
<tr class="odd">
<td style="text-align: left;">192.168.28.254</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:71f3:e219:3f6a:4311</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:9c6b:be4e:1289:2663</td>
</tr>
<tr class="even">
<td style="text-align: left;">2001:dbb:c18:202:a1dd:6355:28ae:da1e</td>
</tr>
<tr class="odd">
<td style="text-align: left;">2001:dbb:c18:202:b9ac:6976:ae1c:e10b</td>
</tr>
<tr class="even">
<td style="text-align: left;">fe80::423c:fcff:fe06:98a4</td>
</tr>
</tbody>
</table>

### Обогащение данных

### Задание 10

#### 10. Определите местоположение (страну, город) и организацию-провайдера для топ-10 доменов. Для этого можно использовать сторонние сервисы, например <https://v4.ifconfig.co/>.
