# Практическая работа №3. Основы обработки данных с помощью R и Dplyr
yaroslavprishedko@yandex.ru

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных
2.  Закрепить знания базовых типов данных языка R
3.  Развить практические навыки использования функций обработки данных
    пакета `dplyr` – функции `select()`, `filter()`, `mutate()`,
    `arrange()`, `group_by()`

## Исходные данные

1.  Программное обеспечение MacOS Version 15.7.1 (24G231)
2.  RStudio Desktop
3.  Интерпретатор языка R 4.5.2
4.  Пакеты `nycflights13` и `dplyr`

## Задание

Используя программный пакет `dplyr`, проанализировать встроенные в пакет
`nycflights13` наборы данных с помощью языка `R` и ответить на вопросы.

## Ход работы

1.  Проанализировать встроенные в пакет nycflights13 наборы данных с
    помощью языка R и ответить на вопросы:
    -   Сколько встроенных в пакет nycflights13 датафреймов?
    -   Сколько строк в каждом датафрейме?
    -   Сколько столбцов в каждом датафрейме?
    -   Как просмотреть примерный вид датафрейма?
    -   Сколько компаний-перевозчиков (carrier) учитывают эти наборы
        данных (представлено в наборах данных)?
    -   Сколько рейсов принял аэропорт John F Kennedy Intl в мае?
    -   Какой самый северный аэропорт?
    -   Какой аэропорт самый высокогорный (находится выше всех над
        уровнем моря)?
    -   Какие бортовые номера у самых старых самолетов?
    -   Какая средняя температура воздуха была в сентябре в аэропорту
        John FKennedy Intl (в градусах Цельсия).
    -   Самолеты какой авиакомпании совершили больше всего вылетов в
        июне?
    -   Самолеты какой авиакомпании задерживались чаще других в 2013
        году?
2.  Оценить результаты и сделать вывод

## Шаги:

### Шаг 1

#### 1. Установка и загрузка необходимых пакетов

    > install.packages('nycflights13')
    trying URL 'https://cran.rstudio.com/bin/macosx/big-sur-arm64/contrib/4.5/nycflights13_1.0.2.tgz'
    Content type 'application/x-gzip' length 4504016 bytes (4.3 MB)
    ==================================================
    downloaded 4.3 MB


    The downloaded binary packages are in
        /var/folders/dq/3s4jxhqs721_fwv6mbjvr9zc0000gn/T//RtmpoKz8jp/downloaded_packages
    > 
    > library(nycflights13)
    > library(dplyr)

#### Вопрос 1: Сколько встроенных в пакет nycflights13 датафреймов?

    > package_data <- data(package = "nycflights13")
    > dataframes <- package_data$results[, "Item"]
    > length(dataframes)
    [1] 5
    > 

#### Вопрос 2: Сколько строк в каждом датафрейме?

    > nrow(airlines)
    [1] 16
    > 
    > 
    > nrow(airports) 
    [1] 1458
    > 
    > 
    > nrow(flights)
    [1] 336776
    > 
    > 
    > nrow(planes)
    [1] 3322
    > 
    > 
    > nrow(weather)
    [1] 26115
    > 
    > 

#### Вопрос 3: Сколько столбцов в каждом датафрейме?

    > ncol(airlines)
    [1] 2
    > 
    > 
    > ncol(airports)
    [1] 8
    > 
    > 
    > ncol(flights) 
    [1] 19
    > 
    > 
    > ncol(planes)
    [1] 9
    > 
    > 
    > ncol(weather)
    [1] 15
    > 
    > 

#### Вопрос 4: Как просмотреть примерный вид датафрейма?

    > head(airlines)
    # A tibble: 6 × 2
      carrier name                    
      <chr>   <chr>                   
    1 9E      Endeavor Air Inc.       
    2 AA      American Airlines Inc.  
    3 AS      Alaska Airlines Inc.    
    4 B6      JetBlue Airways         
    5 DL      Delta Air Lines Inc.    
    6 EV      ExpressJet Airlines Inc.
    > 

#### Вопрос 5: Сколько компаний-перевозчиков (carrier) учитывают эти наборы данных?

    > length(unique(airlines$carrier))
    [1] 16

#### Вопрос 6: Сколько рейсов принял аэропорт John F Kennedy Intl в мае?

    > nrow(flights[flights$dest == "JFK" & flights$month == 5, ])
    [1] 0

#### Вопрос 7: Какой самый северный аэропорт?

    > top_airport <- airports[which.max(airports$lat), ]
    > selected_data <- top_airport[c("name", "lat")]
    > selected_data
    # A tibble: 1 × 2
      name                      lat
      <chr>                   <dbl>
    1 Dillant Hopkins Airport  72.3
    > 
    > 

#### Вопрос 8: Какой аэропорт самый высокогорный (находится выше всех над уровнем моря)?

    > top_airport <- airports[which.max(airports$alt), ]
    > selected_data <- top_airport[c("name", "alt")]
    > 
    > selected_data <- top_airport[c("name", "alt")]
    > selected_data
    # A tibble: 1 × 2
      name        alt
      <chr>     <dbl>
    1 Telluride  9078

#### Вопрос 9: Какие бортовые номера у самых старых самолетов?

    > planes_no_na <- planes[!is.na(planes$year), ]
    > 
    > sorted_planes <- planes_no_na[order(planes_no_na$year), ]
    > 
    > top_10 <- sorted_planes[1:10, ]
    > result <- top_10[c("year", "tailnum")]
    > result
    # A tibble: 10 × 2
        year tailnum
       <int> <chr>  
     1  1956 N381AA 
     2  1959 N201AA 
     3  1959 N567AA 
     4  1963 N378AA 
     5  1963 N575AA 
     6  1965 N14629 
     7  1967 N615AA 
     8  1968 N425AA 
     9  1972 N383AA 
    10  1973 N364AA 
    > 

#### Вопрос 10: Какая средняя температура воздуха была в сентябре в аэропорту John F Kennedy Intl (в градусах Цельсия)?

    > jfk_september <- weather[weather$origin == "JFK" & weather$month == 9, ]
    > 
    > mean_temp_f <- mean(jfk_september$temp, na.rm = TRUE)
    > mean_temp_c <- (mean_temp_f - 32) * 5/9
    > mean_temp_c
    [1] 19.38764

#### Вопрос 11: Самолеты какой авиакомпании совершили больше всего вылетов в июне?

    > june_flights <- flights[flights$month == 6, ]
    > carrier_counts <- table(june_flights$carrier)
    > sorted_carriers <- sort(carrier_counts, decreasing = TRUE)
    > top_carrier <- names(sorted_carriers)[1]
    > count_value <- sorted_carriers[1]
    > airline_name <- airlines[airlines$carrier == top_carrier, "name"]
    > data.frame(name = airline_name, n = count_value)
                        name    n
    UA United Air Lines Inc. 4975

#### Вопрос 12: Самолеты какой авиакомпании задерживались чаще других в 2013 году?

    > complete_flights <- flights[!is.na(flights$dep_delay) & !is.na(flights$arr_delay), ]
    > complete_flights$delayed <- complete_flights$dep_delay > 0 | complete_flights$arr_delay > 0
    > carrier_stats <- aggregate(
    +     cbind(delayed_flights = delayed, total_flights = delayed/delayed) ~ carrier, 
    +     data = complete_flights, 
    +     FUN = function(x) if(is.logical(x)) sum(x) else length(x)
    + )
    > carrier_stats$delay_ratio <- carrier_stats$delayed_flights / carrier_stats$total_flights
    > 
    > carrier_stats <- carrier_stats[order(-carrier_stats$delay_ratio), ]
    > 
    > result <- merge(carrier_stats, airlines, by = "carrier", all.x = TRUE)
    > result
       carrier delayed_flights total_flights delay_ratio                        name
    1       9E            8562          8562           1           Endeavor Air Inc.
    2       AA           14143         14143           1      American Airlines Inc.
    3       AS             289           289           1        Alaska Airlines Inc.
    4       B6           28545         28545           1             JetBlue Airways
    5       DL           21473         21473           1        Delta Air Lines Inc.
    6       EV           28277         28277           1    ExpressJet Airlines Inc.
    7       F9             476           476           1      Frontier Airlines Inc.
    8       FL            2156          2156           1 AirTran Airways Corporation
    9       HA             129           129           1      Hawaiian Airlines Inc.
    10      MQ           12715         12715           1                   Envoy Air
    11      OO              11            11           1       SkyWest Airlines Inc.
    12      UA           32741         32741           1       United Air Lines Inc.
    13      US            8346          8346           1             US Airways Inc.
    14      VX            2745          2745           1              Virgin America
    15      WN            7546          7546           1      Southwest Airlines Co.
    16      YV             292           292           1          Mesa Airlines Inc.
    > 

### Шаг 2

#### Оценка результатов

В ходе выполнения работы были успешно проанализированы наборы данных
пакета `nycflights13` с использованием функций `dplyr` и базовых
операций R.

#### Вывод

В результате выполнения “Практической работы №3” были закреплены навыки
использования языка `R` и пакета `dplyr` для анализа реальных наборов
данных о воздушных перевозках.
