# Практическая работа №5. Исследование информации о состоянии беспроводных сетей
yaroslavprishedko@yandex.ru

## Цель работы

1. Получить знания о методах исследования радиоэлектронной обстановки.
2. Составить представление о механизмах работы Wi-Fi сетей на канальном и сетевом уровне модели OSI.
3. Закрепить практические навыки использования языка программирования R для обработки данных
4. Закрепить знания основных функций обработки данных экосистемы `tidyverse` языка R

## Исходные данные

1.  Программное обеспечение MacOS Version 15.7.1 (24G231)
2.  RStudio Desktop
3.  Интерпретатор языка R 4.5.2

## Общая ситуация

Вы исследуете состояние радиоэлектронной обстановки с помощью журналов программных средств анализа беспроводных сетей – `tcpdump` и
`airodump-ng`. Для этого с помощью сниффера (микрокомпьютера Raspberry
Pi и специализированного Wi-Fi адаптера, переведенного в режим
мониторинга) собирались данные. Сниффер беспроводного трафика был
установлен стационарно (не перемещался). Какой анализ можно провести с
помощью собранной информации

## Задание

Используя программный пакет `dplyr` языка программирования R провести
анализ журналов и ответить на вопросы

## Подготовка к выполнению задания

Произведем загрузку библиотек:

``` r
library(tidyverse)
```

    ── Attaching core tidyverse packages ──────────────────────── tidyverse 2.0.0 ──
    ✔ dplyr     1.1.4     ✔ readr     2.1.6
    ✔ forcats   1.0.1     ✔ stringr   1.6.0
    ✔ ggplot2   4.0.1     ✔ tibble    3.3.0
    ✔ lubridate 1.9.4     ✔ tidyr     1.3.1
    ✔ purrr     1.2.0     
    ── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
    ✖ dplyr::filter() masks stats::filter()
    ✖ dplyr::lag()    masks stats::lag()
    ℹ Use the conflicted package (<http://conflicted.r-lib.org/>) to force all conflicts to become errors

``` r
library(readr)
library(httr)
library(dplyr)
```

## Шаги

### Подготовка данных

#### 1. Импортируйте данные

Для начала скачаем данные:

``` r
if (!dir.exists("data")) {
    dir.create("data")
}

source_url <- "https://storage.yandexcloud.net/dataset.ctfsec/P2_wifi_data.csv"
local_file <- "data/P2_wifi_data.csv"

download.file(
    url = source_url,
    destfile = local_file,
    mode = "wb",
    quiet = FALSE
)
```

Прочитаем данные двух датасетов из CSV-файла:

``` r
file_lines <- read_lines(local_file)
table_boundary <- which(grepl("Station MAC", file_lines))

access_points_data <- read_csv(
    local_file, 
    n_max = table_boundary[1] - 4
)
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
head(access_points_data)
```

    # A tibble: 6 × 15
      BSSID     `First time seen`   `Last time seen`    channel Speed Privacy Cipher
      <chr>     <dttm>              <dttm>                <dbl> <dbl> <chr>   <chr> 
    1 BE:F1:71… 2023-07-28 09:13:03 2023-07-28 11:50:50       1   195 WPA2    CCMP  
    2 6E:C7:EC… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
    3 9A:75:A8… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
    4 4A:EC:1E… 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360 WPA2    CCMP  
    5 D2:6D:52… 2023-07-28 09:13:03 2023-07-28 10:30:19       6   130 WPA2    CCMP  
    6 E8:28:C1… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
    # ℹ 8 more variables: Authentication <chr>, Power <dbl>, `# beacons` <dbl>,
    #   `# IV` <dbl>, `LAN IP` <chr>, `ID-length` <dbl>, ESSID <chr>, Key <lgl>

``` r
stations_data <- read_csv(
    local_file, 
    skip = table_boundary[1] - 2
)
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

``` r
head(stations_data)
```

    # A tibble: 6 × 7
      `Station MAC`  `First time seen`   `Last time seen`    Power `# packets` BSSID
      <chr>          <dttm>              <dttm>              <dbl>       <dbl> <chr>
    1 CA:66:3B:8F:5… 2023-07-28 09:13:03 2023-07-28 10:59:44   -33         858 BE:F…
    2 96:35:2D:3D:8… 2023-07-28 09:13:03 2023-07-28 09:13:03   -65           4 (not…
    3 5C:3A:45:9E:1… 2023-07-28 09:13:03 2023-07-28 11:51:54   -39         432 BE:F…
    4 C0:E4:34:D8:E… 2023-07-28 09:13:03 2023-07-28 11:53:16   -61         958 BE:F…
    5 5E:8E:A6:5E:3… 2023-07-28 09:13:04 2023-07-28 09:13:04   -53           1 (not…
    6 10:51:07:CB:3… 2023-07-28 09:13:05 2023-07-28 11:56:06   -43         344 (not…
    # ℹ 1 more variable: `Probed ESSIDs` <chr>

#### 2. Привести датасеты в вид “аккуратных данных”, преобразовать типы столбцов всоответствии с типом данных

Преобразуем данные

``` r
# Функция для нормализации имен колонок
normalize_column_names <- function(df) {
    names(df) <- names(df) %>%
        gsub("[.# ]", "_", .) %>%
        gsub("_+", "_", .) %>%
        tolower() %>%
        gsub("^_+", "", .) %>%
        gsub("-", "_", .)
    return(df)
}

# Обработка данных точек доступа
normalized_ap_data <- normalize_column_names(access_points_data)

ap_data_final <- normalized_ap_data %>%
    mutate(
        channel = as.integer(channel),
        speed = as.integer(speed),
        power = as.integer(power),
        beacons = as.integer(beacons),
        iv = as.integer(iv),
        id_length = as.integer(id_length),
        privacy = as.factor(privacy),
        cipher = as.factor(cipher),
        authentication = as.factor(authentication)
    )

# Обработка данных клиентов
normalized_stations_data <- normalize_column_names(stations_data)

stations_data_final <- normalized_stations_data %>%
    mutate(
        power = as.integer(power),
        packets = as.integer(packets)
    )
```

#### 3. Просмотрите общую структуру данных с помощью функции `glimpse`

``` r
glimpse(ap_data_final)
```

    Rows: 167
    Columns: 15
    $ bssid           <chr> "BE:F1:71:D5:17:8B", "6E:C7:EC:16:DA:1A", "9A:75:A8:B9…
    $ first_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ last_time_seen  <dttm> 2023-07-28 11:50:50, 2023-07-28 11:55:12, 2023-07-28 …
    $ channel         <int> 1, 1, 1, 7, 6, 6, 11, 11, 11, 1, 6, 14, 11, 11, 6, 6, …
    $ speed           <int> 195, 130, 360, 360, 130, 130, 195, 130, 130, 195, 180,…
    $ privacy         <fct> WPA2, WPA2, WPA2, WPA2, WPA2, OPN, WPA2, WPA2, WPA2, W…
    $ cipher          <fct> CCMP, CCMP, CCMP, CCMP, CCMP, NA, CCMP, CCMP, CCMP, CC…
    $ authentication  <fct> PSK, PSK, PSK, PSK, PSK, NA, PSK, PSK, PSK, PSK, PSK, …
    $ power           <int> -30, -30, -68, -37, -57, -63, -27, -38, -38, -66, -42,…
    $ beacons         <int> 846, 750, 694, 510, 647, 251, 1647, 1251, 704, 617, 13…
    $ iv              <int> 504, 116, 26, 21, 6, 3430, 80, 11, 0, 0, 86, 0, 0, 0, …
    $ lan_ip          <chr> "0.  0.  0.  0", "0.  0.  0.  0", "0.  0.  0.  0", "0.…
    $ id_length       <int> 12, 4, 2, 14, 25, 13, 12, 13, 24, 12, 10, 0, 24, 24, 1…
    $ essid           <chr> "C322U13 3965", "Cnet", "KC", "POCO X5 Pro 5G", NA, "M…
    $ key             <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA…

``` r
glimpse(stations_data_final)
```

    Rows: 12,081
    Columns: 7
    $ station_mac     <chr> "CA:66:3B:8F:56:DD", "96:35:2D:3D:85:E6", "5C:3A:45:9E…
    $ first_time_seen <dttm> 2023-07-28 09:13:03, 2023-07-28 09:13:03, 2023-07-28 …
    $ last_time_seen  <dttm> 2023-07-28 10:59:44, 2023-07-28 09:13:03, 2023-07-28 …
    $ power           <int> -33, -65, -39, -61, -53, -43, -31, -71, -74, -65, -45,…
    $ packets         <int> 858, 4, 432, 958, 1, 344, 163, 3, 115, 437, 265, 77, 7…
    $ bssid           <chr> "BE:F1:71:D5:17:8B", "(not associated)", "BE:F1:71:D6:…
    $ probed_essids   <chr> "C322U13 3965", "IT2 Wireless", "C322U21 0566", "C322U…

### Анализ точек доступа

#### 1. Определить небезопасные точки доступа (без шифрования – OPN)

``` r
unsafe_networks <- ap_data_final[ap_data_final$privacy == "OPN", ]
unsafe_networks
```

    # A tibble: 85 × 15
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     4 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
     5 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     6 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     7 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:27:06       6   130 OPN     <NA>  
     8 E8:28:C… 2023-07-28 09:13:13 2023-07-28 10:39:43       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:17 2023-07-28 11:52:32       1   130 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:13:50 2023-07-28 11:43:39      11   130 OPN     <NA>  
    # ℹ 75 more rows
    # ℹ 8 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>

#### 2. Определить производителя для каждого обнаруженного устройства

Для этого воспользуемся запросами в API, а также настроим кэш, для
уменьшения количества сетевых запросов

``` r
vendor_cache <- new.env(hash = TRUE)

identify_vendor <- function(mac) {
    clean_mac <- gsub("[:.-]", "", mac)
    clean_mac <- toupper(clean_mac)
    
    oui_prefix <- substr(clean_mac, 1, 6)
    
    if (exists(oui_prefix, envir = vendor_cache)) {
        return(get(oui_prefix, envir = vendor_cache))
    }
    
    api_endpoint <- paste0("https://www.macvendorlookup.com/api/v2/", oui_prefix)
    response <- GET(api_endpoint, timeout(3))
    
    if (status_code(response) == 200) {
        parsed_data <- content(response, "parsed")
        
        if (length(parsed_data) > 0 && !is.null(parsed_data[[1]]$company)) {
            company_name <- parsed_data[[1]]$company
            if (company_name == "") {
                company_name <- "Неизвестный производитель"
            }
        } else {
            company_name <- "Неизвестный производитель"
        }
        
        assign(oui_prefix, company_name, envir = vendor_cache)
        return(company_name)
    } else {
        assign(oui_prefix, "Неизвестный производитель", envir = vendor_cache)
        return("Неизвестный производитель")
    }
}

ap_data_enhanced <- ap_data_final
ap_data_enhanced$manufacturer <- sapply(ap_data_enhanced$bssid, identify_vendor)

ap_data_enhanced[, c("bssid", "manufacturer")]
```

    # A tibble: 167 × 2
       bssid             manufacturer             
       <chr>             <chr>                    
     1 BE:F1:71:D5:17:8B Неизвестный производитель
     2 6E:C7:EC:16:DA:1A Неизвестный производитель
     3 9A:75:A8:B9:04:1E Неизвестный производитель
     4 4A:EC:1E:DB:BF:95 Неизвестный производитель
     5 D2:6D:52:61:51:5D Неизвестный производитель
     6 E8:28:C1:DC:B2:52 Eltex Enterprise Ltd.    
     7 BE:F1:71:D6:10:D7 Неизвестный производитель
     8 0A:C5:E1:DB:17:7B Неизвестный производитель
     9 38:1A:52:0D:84:D7 Seiko Epson Corporation  
    10 BE:F1:71:D5:0E:53 Неизвестный производитель
    # ℹ 157 more rows

#### 3. Выявить устройства, использующие последнюю версию протокола шифрования WPA3, и названия точек доступа, реализованных на этих устройствах

``` r
wpa3_networks <- ap_data_enhanced[grep("WPA3", ap_data_enhanced$privacy), ]
wpa3_networks_sorted <- wpa3_networks[order(-wpa3_networks$power), ]
wpa3_networks_sorted
```

    # A tibble: 8 × 16
      bssid     first_time_seen     last_time_seen      channel speed privacy cipher
      <chr>     <dttm>              <dttm>                <int> <int> <fct>   <fct> 
    1 76:C5:A0… 2023-07-28 11:16:36 2023-07-28 11:16:38       6   130 WPA3 W… CCMP  
    2 BE:FD:EF… 2023-07-28 10:15:24 2023-07-28 10:15:28       6   130 WPA3 W… CCMP  
    3 CE:48:E7… 2023-07-28 09:59:20 2023-07-28 10:04:15      44   866 WPA3 W… CCMP  
    4 3A:DA:00… 2023-07-28 10:27:01 2023-07-28 10:27:10       6   130 WPA3 W… CCMP  
    5 8E:1F:94… 2023-07-28 10:08:32 2023-07-28 10:15:27      44   866 WPA3 W… CCMP  
    6 A2:FE:FF… 2023-07-28 09:41:52 2023-07-28 09:41:52       6   130 WPA3 W… CCMP  
    7 26:20:53… 2023-07-28 09:15:45 2023-07-28 09:33:10      44   866 WPA3 W… CCMP  
    8 96:FF:FC… 2023-07-28 09:52:54 2023-07-28 10:25:02      44   866 WPA3 W… CCMP  
    # ℹ 9 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>

#### 4. Отсортировать точки доступа по интервалу времени, в течение которого они находились на связи, по убыванию (Не забудьте склеить сессии! Сессии считаются независимыми если интервал времени между ними превышает 45 минут.)

Для этого реализуем функцию склейки:

``` r
merge_access_point_sessions <- function(data, gap_threshold = 45) {
    data$first_time_seen <- as.POSIXct(data$first_time_seen)
    data$last_time_seen <- as.POSIXct(data$last_time_seen)
    
    data <- data[order(data$bssid, data$first_time_seen), ]
    
    process_single_bssid <- function(bssid_group) {
        if (nrow(bssid_group) <= 1) {
            bssid_group$session_id <- 1
            bssid_group$session_duration_minutes <- as.numeric(
                difftime(bssid_group$last_time_seen, bssid_group$first_time_seen, units = "mins")
            )
            return(bssid_group)
        }
        
        bssid_group <- bssid_group[order(bssid_group$first_time_seen), ]
        
        bssid_group$prev_end_time <- c(NA, bssid_group$last_time_seen[-nrow(bssid_group)])
        bssid_group$time_gap <- as.numeric(
            difftime(bssid_group$first_time_seen, bssid_group$prev_end_time, units = "mins")
        )
        
        bssid_group$new_session <- c(TRUE, bssid_group$time_gap[-1] > gap_threshold)
        bssid_group$session_id <- cumsum(bssid_group$new_session)
        
        merged_sessions <- aggregate(
            cbind(
                first_time_seen = first_time_seen,
                last_time_seen = last_time_seen,
                beacons = beacons,
                iv = iv
            ) ~ session_id + essid + channel + privacy + manufacturer,
            data = bssid_group,
            FUN = function(x) if (is.POSIXct(x[1])) min(x) else if (is.POSIXct(x[2])) max(x) else sum(x, na.rm = TRUE)
        )
        
        merged_sessions$session_duration_minutes <- as.numeric(
            difftime(merged_sessions$last_time_seen, merged_sessions$first_time_seen, units = "mins")
        )
        
        merged_sessions$session_duration_hours <- round(merged_sessions$session_duration_minutes / 60, 2)
        merged_sessions$session_duration_days <- round(merged_sessions$session_duration_minutes / (60 * 24), 2)
        
        return(merged_sessions)
    }
    
    result_list <- by(data, data$bssid, process_single_bssid)
    combined_result <- do.call(rbind, result_list)
    rownames(combined_result) <- NULL
    
    combined_result <- combined_result[order(-combined_result$session_duration_minutes), ]
    
    return(combined_result)
}

session_results <- merge_access_point_sessions(ap_data_enhanced)
session_results
```

    # A tibble: 167 × 18
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 00:25:0… 2023-07-28 09:13:06 2023-07-28 11:56:21      44    -1 OPN     <NA>  
     2 E8:28:C… 2023-07-28 09:13:09 2023-07-28 11:56:05      11   130 OPN     <NA>  
     3 E8:28:C… 2023-07-28 09:13:03 2023-07-28 11:55:38       6   130 OPN     <NA>  
     4 08:3A:2… 2023-07-28 09:13:27 2023-07-28 11:55:53      14    -1 WPA     <NA>  
     5 6E:C7:E… 2023-07-28 09:13:03 2023-07-28 11:55:12       1   130 WPA2    CCMP  
     6 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:12       6   130 OPN     <NA>  
     7 48:5B:3… 2023-07-28 09:13:06 2023-07-28 11:55:11       1   270 WPA2    CCMP  
     8 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:11       6   130 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:13:06 2023-07-28 11:55:10       6    -1 OPN     <NA>  
    10 8E:55:4… 2023-07-28 09:13:06 2023-07-28 11:55:09       6    65 WPA2    CCMP  
    # ℹ 157 more rows
    # ℹ 11 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>, session_id <dbl>, session_duration_minutes <dbl>

#### 5. Обнаружить топ-10 самых быстрых точек доступа

``` r
fast_networks <- ap_data_enhanced[!is.na(ap_data_enhanced$speed) & ap_data_enhanced$speed > 0, ]

top_speed_networks <- fast_networks[order(-fast_networks$speed), ][1:10, ]
top_speed_networks
```

    # A tibble: 10 × 16
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 26:20:5… 2023-07-28 09:15:45 2023-07-28 09:33:10      44   866 WPA3 W… CCMP  
     2 96:FF:F… 2023-07-28 09:52:54 2023-07-28 10:25:02      44   866 WPA3 W… CCMP  
     3 CE:48:E… 2023-07-28 09:59:20 2023-07-28 10:04:15      44   866 WPA3 W… CCMP  
     4 8E:1F:9… 2023-07-28 10:08:32 2023-07-28 10:15:27      44   866 WPA3 W… CCMP  
     5 9A:75:A… 2023-07-28 09:13:03 2023-07-28 11:53:31       1   360 WPA2    CCMP  
     6 4A:EC:1… 2023-07-28 09:13:03 2023-07-28 11:04:01       7   360 WPA2    CCMP  
     7 56:C5:2… 2023-07-28 09:17:49 2023-07-28 10:27:22       1   360 WPA2    CCMP  
     8 E8:28:C… 2023-07-28 09:18:16 2023-07-28 11:36:43      48   360 OPN     <NA>  
     9 E8:28:C… 2023-07-28 09:18:16 2023-07-28 11:51:48      48   360 OPN     <NA>  
    10 E8:28:C… 2023-07-28 09:18:30 2023-07-28 11:43:23      48   360 OPN     <NA>  
    # ℹ 9 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>

#### 6. Отсортировать точки доступа по частоте отправки запросов (beacons) в единицу времени по их убыванию

``` r
valid_beacon_data <- ap_data_enhanced[
    !is.na(ap_data_enhanced$beacons) & ap_data_enhanced$beacons > 0 & 
    !is.na(ap_data_enhanced$first_time_seen) & !is.na(ap_data_enhanced$last_time_seen), ]

valid_beacon_data$first_time_seen <- as.POSIXct(valid_beacon_data$first_time_seen)
valid_beacon_data$last_time_seen <- as.POSIXct(valid_beacon_data$last_time_seen)

valid_beacon_data$observation_duration <- as.numeric(
    difftime(valid_beacon_data$last_time_seen, valid_beacon_data$first_time_seen, units = "mins")
)

valid_beacon_data$observation_duration <- ifelse(
    valid_beacon_data$observation_duration <= 0, 0.1, valid_beacon_data$observation_duration
)

valid_beacon_data$beacons_per_minute <- valid_beacon_data$beacons / valid_beacon_data$observation_duration
valid_beacon_data$beacons_per_second <- valid_beacon_data$beacons_per_minute / 60
valid_beacon_data$beacons_per_hour <- valid_beacon_data$beacons / (valid_beacon_data$observation_duration / 60)

beacon_frequency_ranking <- valid_beacon_data[order(-valid_beacon_data$beacons_per_minute), ]
beacon_frequency_ranking
```

    # A tibble: 102 × 20
       bssid    first_time_seen     last_time_seen      channel speed privacy cipher
       <chr>    <dttm>              <dttm>                <int> <int> <fct>   <fct> 
     1 F2:30:A… 2023-07-28 10:27:02 2023-07-28 10:27:09       1   130 WPA2    CCMP  
     2 B2:CF:C… 2023-07-28 10:40:54 2023-07-28 10:40:59       6   180 WPA2    CCMP  
     3 3A:DA:0… 2023-07-28 10:27:01 2023-07-28 10:27:10       6   130 WPA3 W… CCMP  
     4 02:BC:1… 2023-07-28 09:24:46 2023-07-28 09:24:48      11   130 OPN     <NA>  
     5 00:3E:1… 2023-07-28 10:34:03 2023-07-28 10:34:05      11   130 OPN     <NA>  
     6 76:C5:A… 2023-07-28 11:16:36 2023-07-28 11:16:38       6   130 WPA3 W… CCMP  
     7 D2:25:9… 2023-07-28 09:45:29 2023-07-28 09:45:42      12   180 WPA2    CCMP  
     8 BE:F1:7… 2023-07-28 09:13:03 2023-07-28 11:50:44      11   195 WPA2    CCMP  
     9 76:E4:E… 2023-07-28 09:18:50 2023-07-28 09:18:50      13   180 WPA2    CCMP  
    10 C2:B5:D… 2023-07-28 09:32:42 2023-07-28 09:32:42       6   130 WPA2    CCMP  
    # ℹ 92 more rows
    # ℹ 13 more variables: authentication <fct>, power <int>, beacons <int>,
    #   iv <int>, lan_ip <chr>, id_length <int>, essid <chr>, key <lgl>,
    #   manufacturer <chr>, observation_duration <dbl>, beacons_per_minute <dbl>,
    #   beacons_per_second <dbl>, beacons_per_hour <dbl>

### Анализ клиентов

#### 1. Определить производителя для каждого обнаруженного устройства

Для этого воспользуемся функцией, написанной нами для поиска
производителей точек доступа:

``` r
valid_stations <- stations_data_final[
    !is.na(stations_data_final$bssid) & stations_data_final$bssid != "(not associated)", ]

valid_stations$manufacturer <- sapply(valid_stations$bssid, identify_vendor)

valid_stations[, c("bssid", "manufacturer")]
```

    # A tibble: 186 × 2
       bssid             manufacturer                 
       <chr>             <chr>                        
     1 BE:F1:71:D5:17:8B Неизвестный производитель    
     2 BE:F1:71:D6:10:D7 Неизвестный производитель    
     3 BE:F1:71:D5:17:8B Неизвестный производитель    
     4 1E:93:E3:1B:3C:F4 Неизвестный производитель    
     5 E8:28:C1:DC:FF:F2 Eltex Enterprise Ltd.        
     6 00:25:00:FF:94:73 Apple, Inc.                  
     7 00:26:99:F2:7A:E2 Cisco Systems, Inc           
     8 0C:80:63:A9:6E:EE TP-LINK TECHNOLOGIES CO.,LTD.
     9 E8:28:C1:DD:04:52 Eltex Enterprise Ltd.        
    10 0A:C5:E1:DB:17:7B Неизвестный производитель    
    # ℹ 176 more rows

#### 2. Обнаружить устройства, которые НЕ рандомизируют свой MAC адрес

``` r
non_random_indicators <- c("A", "a", "E", "e", "2", "6")

devices_without_randomization <- valid_stations[
    !substr(valid_stations$station_mac, 2, 2) %in% non_random_indicators, ]
devices_without_randomization
```

    # A tibble: 59 × 8
       station_mac       first_time_seen     last_time_seen      power packets bssid
       <chr>             <dttm>              <dttm>              <int>   <int> <chr>
     1 5C:3A:45:9E:1A:7B 2023-07-28 09:13:03 2023-07-28 11:51:54   -39     432 BE:F…
     2 C0:E4:34:D8:E7:E5 2023-07-28 09:13:03 2023-07-28 11:53:16   -61     958 BE:F…
     3 68:54:5A:40:35:9E 2023-07-28 09:13:06 2023-07-28 11:50:50   -31     163 1E:9…
     4 74:4C:A1:70:CE:F7 2023-07-28 09:13:06 2023-07-28 09:20:01   -71       3 E8:2…
     5 4C:44:5B:14:76:E3 2023-07-28 09:13:09 2023-07-28 09:47:44    -1      71 E8:2…
     6 A0:E7:0B:AE:D5:44 2023-07-28 09:13:09 2023-07-28 11:34:42   -37     125 0A:C…
     7 48:68:4A:93:DF:B4 2023-07-28 09:13:13 2023-07-28 11:50:22   -48     122 1E:9…
     8 28:7F:CF:23:25:53 2023-07-28 09:13:14 2023-07-28 11:51:50   -37     156 9A:7…
     9 8C:55:4A:DE:F2:38 2023-07-28 09:13:17 2023-07-28 11:56:16   -65     117 8A:A…
    10 88:D8:2E:4F:9B:1A 2023-07-28 09:13:19 2023-07-28 11:51:24   -29     240 4A:E…
    # ℹ 49 more rows
    # ℹ 2 more variables: probed_essids <chr>, manufacturer <chr>

#### 3. Кластеризовать запросы от устройств к точкам доступа по их именам. Определить время появления устройства в зоне радиовидимости и время выхода его из нее.

``` r
client_data_prepared <- valid_stations
client_data_prepared$first_time_seen <- as.POSIXct(client_data_prepared$first_time_seen)
client_data_prepared$last_time_seen <- as.POSIXct(client_data_prepared$last_time_seen)
client_data_prepared$probed_essids <- ifelse(
    is.na(client_data_prepared$probed_essids) | client_data_prepared$probed_essids == "", 
    "Unknown", 
    as.character(client_data_prepared$probed_essids)
)
client_data_prepared <- client_data_prepared[client_data_prepared$probed_essids != "Unknown", ]

split_essids <- str_split(client_data_prepared$probed_essids, ",")
client_data_expanded <- client_data_prepared[rep(seq_len(nrow(client_data_prepared)), lengths(split_essids)), ]
client_data_expanded$cluster <- trimws(unlist(split_essids))
client_data_expanded <- client_data_expanded[client_data_expanded$cluster != "" & client_data_expanded$cluster != "Unknown", ]

cluster_summary <- aggregate(
    cbind(
        first_appearance = first_time_seen,
        last_appearance = last_time_seen,
        power_values = power,
        packet_counts = packets
    ) ~ station_mac + cluster,
    data = client_data_expanded,
    FUN = function(x) {
        if (is.POSIXct(x[1])) min(x, na.rm = TRUE) else if (is.POSIXct(x[2])) max(x, na.rm = TRUE) else x
    }
)

names(cluster_summary)[names(cluster_summary) == "first_appearance"] <- "first_appearance"
names(cluster_summary)[names(cluster_summary) == "last_appearance"] <- "last_appearance"
names(cluster_summary)[names(cluster_summary) == "power_values"] <- "avg_power"
names(cluster_summary)[names(cluster_summary) == "packet_counts"] <- "total_packets"

cluster_summary$duration_minutes <- as.numeric(
    difftime(cluster_summary$last_appearance, cluster_summary$first_appearance, units = "mins")
)
cluster_summary$packets_per_minute <- ifelse(
    cluster_summary$duration_minutes > 0,
    round(cluster_summary$total_packets / cluster_summary$duration_minutes, 2),
    0
)

cluster_summary <- cluster_summary[order(cluster_summary$station_mac, cluster_summary$first_appearance), ]
cluster_summary
```

              station_mac
    45  00:F4:8D:F7:C5:19
    104 00:F4:8D:F7:C5:19
    27  02:69:A5:29:F1:3E
    65  04:8C:9A:0B:40:EA
    66  06:15:2E:12:C8:A6
    94  06:15:2E:12:C8:A6
    67  06:F2:A9:C1:8D:09
    68  0A:AB:49:39:BB:29
    69  0A:C2:C3:08:9E:F8
    55  0C:E4:41:E8:C3:6E
    31  0E:AD:54:09:04:37
    32  12:49:27:AA:B0:A5
    70  16:9C:FD:6C:63:E1
    114 16:BA:0E:97:29:A4
    71  1A:E3:05:F5:C7:B5
    72  26:57:A7:54:0B:B3
    49  28:7F:CF:23:25:53
    103 28:CD:C4:96:D0:E9
    56  2A:20:99:BE:E5:EF
    33  2A:2C:2C:AB:65:80
    107 2A:BF:50:E5:74:84
    73  32:D4:08:4F:02:F2
    15  34:7D:F6:95:25:E5
    17  34:7D:F6:95:25:E5
    22  34:E1:2D:3C:C8:2D
    51  40:74:E0:99:49:EF
    46  42:5E:24:27:71:03
    115 42:B4:2E:80:E8:4D
    34  42:C7:F5:C1:59:B0
    1   48:2C:A0:74:B7:12
    6   48:2C:A0:74:B7:12
    10  48:2C:A0:74:B7:12
    12  48:2C:A0:74:B7:12
    74  48:2C:A0:74:B7:12
    120 48:2C:A0:74:B7:12
    28  48:68:4A:93:DF:B4
    95  48:74:12:82:D7:91
    52  4A:C9:28:46:EE:3F
    9   4C:E0:DB:0B:F8:C2
    98  4C:E0:DB:0B:F8:C2
    100 4C:E0:DB:0B:F8:C2
    44  4E:98:AA:FF:EE:A7
    53  4E:98:AA:FF:EE:A7
    57  4E:98:AA:FF:EE:A7
    96  4E:98:AA:FF:EE:A7
    99  4E:98:AA:FF:EE:A7
    111 4E:98:AA:FF:EE:A7
    113 4E:98:AA:FF:EE:A7
    35  50:3E:AA:5F:AB:23
    30  50:C2:E8:0A:D9:5F
    21  52:63:E0:44:D2:E3
    50  52:63:E0:44:D2:E3
    75  52:63:E0:44:D2:E3
    93  52:63:E0:44:D2:E3
    116 52:FE:C5:8B:DF:D3
    76  54:71:DD:FE:12:68
    58  56:14:6C:F1:A5:9E
    48  58:6C:25:9E:61:DC
    20  5C:3A:45:9E:1A:7B
    77  5E:54:01:D6:05:1C
    78  5E:C0:A3:79:C7:48
    16  68:54:5A:40:35:9E
    29  68:54:5A:40:35:9E
    59  6A:AE:70:91:03:48
    60  6A:CE:A6:F8:C7:54
    79  70:66:55:D0:B6:C7
    61  72:58:69:DB:59:C7
    62  72:5A:52:B8:93:BF
    63  74:DF:BF:7B:00:19
    80  74:DF:BF:7B:00:19
    36  76:D3:55:13:DB:CC
    25  76:EB:80:88:6A:08
    108 78:2B:46:FF:46:0A
    102 88:D8:2E:4F:9B:1A
    81  8A:CE:15:ED:E5:D1
    3   8A:E8:C5:3D:A3:10
    4   8A:E8:C5:3D:A3:10
    5   8A:E8:C5:3D:A3:10
    7   8A:E8:C5:3D:A3:10
    8   8A:E8:C5:3D:A3:10
    11  8A:E8:C5:3D:A3:10
    82  8A:E8:C5:3D:A3:10
    26  8C:55:4A:DE:F2:38
    83  8C:55:4A:DE:F2:38
    84  98:F6:21:30:3D:31
    13  98:F6:21:72:9E:D6
    85  98:F6:21:72:9E:D6
    112 98:F6:21:72:9E:D6
    37  9A:7D:CE:C9:0A:DF
    38  9A:F7:00:BC:C9:69
    64  9E:29:EA:E9:70:20
    2   A0:E7:0B:AE:D5:44
    86  A2:D2:22:A7:E1:BB
    54  AC:5F:3E:2F:D7:8D
    106 AC:5F:3E:2F:D7:8D
    118 AC:5F:3E:2F:D7:8D
    119 AC:5F:3E:2F:D7:8D
    109 AC:B5:7D:B4:92:05
    39  B0:BE:83:81:8E:68
    23  B8:27:EB:59:FA:0E
    24  BC:F1:71:4A:20:8B
    105 BC:F1:71:4A:20:8B
    18  C0:E4:34:D8:E7:E5
    110 C4:75:AB:E5:6E:EC
    47  C6:49:D0:8D:CF:6C
    40  C6:D1:E8:6A:1C:C3
    87  C6:E4:7B:0F:55:DF
    41  CA:54:C4:8B:B5:3A
    19  CA:66:3B:8F:56:DD
    88  D2:4E:C6:AA:CA:5F
    101 D4:54:8B:8F:56:E7
    97  D6:5F:79:37:0A:01
    89  D8:9B:3B:FA:6D:50
    117 DA:5F:56:E4:CB:E5
    42  DA:B5:28:4A:FC:4D
    14  F0:67:28:61:26:6B
    90  F0:67:28:61:26:6B
    43  F6:4D:98:98:18:C3
    91  F6:96:2D:85:19:81
    92  FA:2A:CD:85:24:5F
                                                                                 cluster
    45                                                                          Hornet24
    104                                                                         Redmi 12
    27                                                                        Galaxy A71
    65                                                                     MIREA_HOTSPOT
    66                                                                     MIREA_HOTSPOT
    94                                                                           MT_FREE
    67                                                                     MIREA_HOTSPOT
    68                                                                     MIREA_HOTSPOT
    69                                                                     MIREA_HOTSPOT
    55                                                                      MIREA_GUESTS
    31                                                                              GIVC
    32                                                                              GIVC
    70                                                                     MIREA_HOTSPOT
    114                                                                         Vladimir
    71                                                                     MIREA_HOTSPOT
    72                                                                     MIREA_HOTSPOT
    49                                                                                KC
    103                                                                        realme 10
    56                                                                      MIREA_GUESTS
    33                                                                              GIVC
    107                                                                 Redmi Note 8 Pro
    73                                                                     MIREA_HOTSPOT
    15                                                                      C322U06 5179
    17                                                                      C322U06 9080
    22                                                                              Cnet
    51                                                                        KOTIKI_XXX
    46                                                                               IKB
    115                                                                         Vladimir
    34                                                                              GIVC
    1   \\xA7\\xDF\\xA7\\xD1\\xA7\\xE3\\xA7\\xE4\\xA7\\xD6\\xA7\\xE9\\xA7\\xDC\\xA7\\xD1
    6                                                                  AndroidShare_3542
    10                                                                 AndroidShare_8795
    12                                                      AQAAAB6zaIoATwEURedmi Note 5
    74                                                                     MIREA_HOTSPOT
    120                                                                         настечка
    28                                                                        Galaxy A71
    95                                                                           MT_FREE
    52                                                                        KOTIKI_XXX
    9                                                                  AndroidShare_8397
    98                                                                   MTS_GPON_8cedb0
    100                                                                netis_2.4G_DABAEE
    44                                                                             Gvrcy
    53                                                                              Kv15
    57                                                                      MIREA_GUESTS
    96                                                                           MT_FREE
    99                                                               MTSRouter_5G_142878
    111                                                                      Sharminginn
    113                                                                  TP-Link_9872_5G
    35                                                                              GIVC
    30                                                                GGWPRedmi Note 10S
    21                                                                      C349U26 3598
    50                                                                           kmkdz_g
    75                                                                     MIREA_HOTSPOT
    93                                                                  Moscow_WiFi_Free
    116                                                                         Vladimir
    76                                                                     MIREA_HOTSPOT
    58                                                                      MIREA_GUESTS
    48                                                                 iPhone (Искандер)
    20                                                                      C322U21 0566
    77                                                                     MIREA_HOTSPOT
    78                                                                     MIREA_HOTSPOT
    16                                                                      C322U06 5179
    29                                                                        Galaxy A71
    59                                                                      MIREA_GUESTS
    60                                                                      MIREA_GUESTS
    79                                                                     MIREA_HOTSPOT
    61                                                                      MIREA_GUESTS
    62                                                                      MIREA_GUESTS
    63                                                                      MIREA_GUESTS
    80                                                                     MIREA_HOTSPOT
    36                                                                              GIVC
    25                                                                DIRECT-03-BMW44597
    108                                                                 Redmi Note 8 Pro
    102                                                                   POCO X5 Pro 5G
    81                                                                     MIREA_HOTSPOT
    3                                                                  AndroidShare_2061
    4                                                                  AndroidShare_2335
    5                                                                  AndroidShare_2800
    7                                                                  AndroidShare_6329
    8                                                                  AndroidShare_7271
    11                                                                 AndroidShare_8927
    82                                                                     MIREA_HOTSPOT
    26                                                                   Galaxy A30s5208
    83                                                                     MIREA_HOTSPOT
    84                                                                     MIREA_HOTSPOT
    13                                                                 Beeline_5G_F2F425
    85                                                                     MIREA_HOTSPOT
    112                                                                    Snickers_ASSA
    37                                                                              GIVC
    38                                                                              GIVC
    64                                                                      MIREA_GUESTS
    2                                                                      AndroidAP177B
    86                                                                     MIREA_HOTSPOT
    54                                                                        lenovo_pad
    106                                                                          Redmi 9
    118                                                                            Wi-Fi
    119                                                                            WiFi2
    109                                                                    Redmi Note 9S
    39                                                                              GIVC
    23                                                                              Cnet
    24                                                                              Damy
    105                                                                         Redmi 12
    18                                                                      C322U13 3965
    110                                                                     SERVICE.live
    47                                                                               IKB
    40                                                                              GIVC
    87                                                                     MIREA_HOTSPOT
    41                                                                              GIVC
    19                                                                      C322U13 3965
    88                                                                     MIREA_HOTSPOT
    101                                                                       OnePlus 6T
    97                                                                           MT_FREE
    89                                                                     MIREA_HOTSPOT
    117                                                                         Vladimir
    42                                                                              GIVC
    14                                                                  Belorusneft Free
    90                                                                     MIREA_HOTSPOT
    43                                                                              GIVC
    91                                                                     MIREA_HOTSPOT
    92                                                                     MIREA_HOTSPOT
        first_appearance last_appearance avg_power total_packets duration_minutes
    45        1690541104      1690544606       -73             8      58.36666667
    104       1690541104      1690544606       -73             8      58.36666667
    27        1690541555      1690543491       -49            26      32.26666667
    65        1690540067      1690545351       -73           756      88.06666667
    66        1690540067      1690540241       -63            12       2.90000000
    94        1690540067      1690540241       -63            12       2.90000000
    67        1690536700      1690545058       -73           142     139.30000000
    68        1690540359      1690544384       -61           801      67.08333333
    69        1690539279      1690539287       -59             9       0.13333333
    55        1690536727      1690539700       -86            48      49.55000000
    31        1690535631      1690537402       -71            16      29.51666667
    32        1690542132      1690544942       -57             8      46.83333333
    70        1690540474      1690540487       -67           223       0.21666667
    114       1690539443      1690539742       -51            34       4.98333333
    71        1690536161      1690536185       -61            30       0.40000000
    72        1690535724      1690539723       -59            14      66.65000000
    49        1690535594      1690545110       -37           156     158.60000000
    103       1690536591      1690539955       -59           359      56.06666667
    56        1690543735      1690543771       -61            11       0.60000000
    33        1690536281      1690545330       -67            28     150.81666667
    107       1690540676      1690545175       -55            26      74.98333333
    73        1690540778      1690540787       -55            22       0.15000000
    15        1690535600      1690539619       -43            76      66.98333333
    17        1690535600      1690539619       -43            76      66.98333333
    22        1690535609      1690545110       -59           580     158.35000000
    51        1690537132      1690545216       -85            23     134.73333333
    46        1690539649      1690540432       -53            52      13.05000000
    115       1690537139      1690541219       -51            35      68.00000000
    34        1690536230      1690545193       -67           114     149.38333333
    1         1690539919      1690545108       -82           134      86.48333333
    6         1690539919      1690545108       -82           134      86.48333333
    10        1690539919      1690545108       -82           134      86.48333333
    12        1690539919      1690545108       -82           134      86.48333333
    74        1690539919      1690545108       -82           134      86.48333333
    120       1690539919      1690545108       -82           134      86.48333333
    28        1690535593      1690545022       -48           122     157.15000000
    95        1690536037      1690545078       -69            55     150.68333333
    52        1690535588      1690544189       -65            77     143.35000000
    9         1690539899      1690539904       -61           114       0.08333333
    98        1690539899      1690539904       -61           114       0.08333333
    100       1690539899      1690539904       -61           114       0.08333333
    44        1690540290      1690540322       -63            21       0.53333333
    53        1690540290      1690540322       -63            21       0.53333333
    57        1690540290      1690540322       -63            21       0.53333333
    96        1690540290      1690540322       -63            21       0.53333333
    99        1690540290      1690540322       -63            21       0.53333333
    111       1690540290      1690540322       -63            21       0.53333333
    113       1690540290      1690540322       -63            21       0.53333333
    35        1690537142      1690544799       -61            90     127.61666667
    30        1690544084      1690545108       -57            35      17.06666667
    21        1690536785      1690536813       -23            47       0.46666667
    50        1690536785      1690536813       -23            47       0.46666667
    75        1690536785      1690536813       -23            47       0.46666667
    93        1690536785      1690536813       -23            47       0.46666667
    116       1690536717      1690545247       -57          1037     142.16666667
    76        1690539949      1690540457       -67             3       8.46666667
    58        1690541489      1690541492       -57           116       0.05000000
    48        1690536062      1690544598       -42           158     142.26666667
    20        1690535583      1690545114       -39           432     158.85000000
    77        1690540111      1690540502       -45           135       6.51666667
    78        1690539003      1690539070       -63            12       1.11666667
    16        1690535586      1690545050       -31           163     157.73333333
    29        1690535586      1690545050       -31           163     157.73333333
    59        1690540036      1690540049       -63            12       0.21666667
    60        1690540602      1690540671       -69           205       1.15000000
    79        1690535649      1690545381       -83           184     162.20000000
    61        1690540239      1690540244       -69            12       0.08333333
    62        1690542468      1690544859       -51           337      39.85000000
    63        1690537548      1690540271       -65           911      45.38333333
    80        1690537548      1690540271       -65           911      45.38333333
    36        1690544380      1690544380       -51             2       0.00000000
    25        1690536595      1690536595       -83             2       0.00000000
    108       1690535984      1690545183       -43           371     153.31666667
    102       1690535599      1690545084       -29           240     158.08333333
    81        1690535976      1690536066       -47            20       1.50000000
    3         1690540147      1690540180       -67            31       0.55000000
    4         1690540147      1690540180       -67            31       0.55000000
    5         1690540147      1690540180       -67            31       0.55000000
    7         1690540147      1690540180       -67            31       0.55000000
    8         1690540147      1690540180       -67            31       0.55000000
    11        1690540147      1690540180       -67            31       0.55000000
    82        1690540147      1690540180       -67            31       0.55000000
    26        1690535597      1690545376       -65           117     162.98333333
    83        1690535597      1690545376       -65           117     162.98333333
    84        1690539289      1690543811       -63           142      75.36666667
    13        1690540875      1690545374       -59          2143      74.98333333
    85        1690540875      1690545374       -59          2143      74.98333333
    112       1690540875      1690545374       -59          2143      74.98333333
    37        1690541587      1690545015       -59            29      57.13333333
    38        1690540116      1690545354       -61           358      87.30000000
    64        1690540502      1690540505       -61             7       0.05000000
    2         1690535589      1690544082       -37           125     141.55000000
    86        1690540223      1690540236       -69            15       0.21666667
    54        1690542316      1690545028       -69            43      45.20000000
    106       1690542316      1690545028       -69            43      45.20000000
    118       1690542316      1690545028       -69            43      45.20000000
    119       1690542316      1690545028       -69            43      45.20000000
    109       1690535631      1690544030       -73           281     139.98333333
    39        1690540526      1690545336       -63           430      80.16666667
    23        1690535616      1690544644        -1           405     150.46666667
    24        1690540121      1690545044       -84            44      82.05000000
    105       1690540121      1690545044       -84            44      82.05000000
    18        1690535583      1690545196       -61           958     160.21666667
    110       1690540272      1690545334       -63            15      84.36666667
    47        1690539648      1690540445       -59            28      13.28333333
    40        1690540116      1690540282       -67            60       2.76666667
    87        1690540360      1690540482       -45             8       2.03333333
    41        1690535586      1690545304       -65           437     161.96666667
    19        1690535583      1690541984       -33           858     106.68333333
    88        1690536896      1690544679       -71            70     129.71666667
    101       1690535837      1690539277       -49            69      57.33333333
    97        1690543616      1690543616       -67             1       0.00000000
    89        1690540531      1690545309       -61           498      79.63333333
    117       1690536909      1690541555       -47           194      77.43333333
    42        1690545163      1690545275       -65             9       1.86666667
    14        1690535863      1690543495       -67            89     127.20000000
    90        1690535863      1690543495       -67            89     127.20000000
    43        1690535677      1690545329       -61          1062     160.86666667
    91        1690544833      1690544839       -61            35       0.10000000
    92        1690540318      1690540327       -65            61       0.15000000
        packets_per_minute
    45                0.14
    104               0.14
    27                0.81
    65                8.58
    66                4.14
    94                4.14
    67                1.02
    68               11.94
    69               67.50
    55                0.97
    31                0.54
    32                0.17
    70             1029.23
    114               6.82
    71               75.00
    72                0.21
    49                0.98
    103               6.40
    56               18.33
    33                0.19
    107               0.35
    73              146.67
    15                1.13
    17                1.13
    22                3.66
    51                0.17
    46                3.98
    115               0.51
    34                0.76
    1                 1.55
    6                 1.55
    10                1.55
    12                1.55
    74                1.55
    120               1.55
    28                0.78
    95                0.37
    52                0.54
    9              1368.00
    98             1368.00
    100            1368.00
    44               39.38
    53               39.38
    57               39.38
    96               39.38
    99               39.38
    111              39.38
    113              39.38
    35                0.71
    30                2.05
    21              100.71
    50              100.71
    75              100.71
    93              100.71
    116               7.29
    76                0.35
    58             2320.00
    48                1.11
    20                2.72
    77               20.72
    78               10.75
    16                1.03
    29                1.03
    59               55.38
    60              178.26
    79                1.13
    61              144.00
    62                8.46
    63               20.07
    80               20.07
    36                0.00
    25                0.00
    108               2.42
    102               1.52
    81               13.33
    3                56.36
    4                56.36
    5                56.36
    7                56.36
    8                56.36
    11               56.36
    82               56.36
    26                0.72
    83                0.72
    84                1.88
    13               28.58
    85               28.58
    112              28.58
    37                0.51
    38                4.10
    64              140.00
    2                 0.88
    86               69.23
    54                0.95
    106               0.95
    118               0.95
    119               0.95
    109               2.01
    39                5.36
    23                2.69
    24                0.54
    105               0.54
    18                5.98
    110               0.18
    47                2.11
    40               21.69
    87                3.93
    41                2.70
    19                8.04
    88                0.54
    101               1.20
    97                0.00
    89                6.25
    117               2.51
    42                4.82
    14                0.70
    90                0.70
    43                6.60
    91              350.00
    92              406.67

#### 4. Оценить стабильность уровня сигнала внури кластера во времени. Выявить наиболее стабильный кластер.

``` r
stability_metrics <- aggregate(
    power ~ cluster,
    data = client_data_expanded,
    FUN = function(x) {
        c(
            request_count = length(x),
            unique_devices = length(unique(client_data_expanded$station_mac[client_data_expanded$cluster == unique(client_data_expanded$cluster[match(x, client_data_expanded$power)])])),
            mean_signal = round(mean(x, na.rm = TRUE), 2),
            median_signal = round(median(x, na.rm = TRUE), 2),
            std_deviation = if (length(x) == 1) 0 else round(sd(x, na.rm = TRUE), 2),
            min_signal = min(x, na.rm = TRUE),
            max_signal = max(x, na.rm = TRUE)
        )
    }
)
```

    Warning in client_data_expanded$cluster ==
    unique(client_data_expanded$cluster[match(x, : longer object length is not a
    multiple of shorter object length

``` r
stability_analysis <- data.frame(
    cluster = stability_metrics$cluster,
    n_requests = sapply(stability_metrics$power, function(x) x[1]),
    unique_devices = sapply(stability_metrics$power, function(x) x[2]),
    mean_power = sapply(stability_metrics$power, function(x) x[3]),
    median_power = sapply(stability_metrics$power, function(x) x[4]),
    sd_power = sapply(stability_metrics$power, function(x) x[5]),
    min_power = sapply(stability_metrics$power, function(x) x[6]),
    max_power = sapply(stability_metrics$power, function(x) x[7])
)

stability_analysis$range_power <- round(stability_analysis$max_power - stability_analysis$min_power, 2)
stability_analysis$cv_power <- ifelse(
    abs(stability_analysis$mean_power) > 0,
    round((stability_analysis$sd_power / abs(stability_analysis$mean_power)) * 100, 2),
    0
)

filtered_stability <- stability_analysis[stability_analysis$n_requests >= 5, ]

filtered_stability <- filtered_stability[order(filtered_stability$sd_power), ]

most_stable_network <- filtered_stability[1, ]
most_stable_network
```

       cluster n_requests unique_devices mean_power median_power sd_power min_power
    26    GIVC         13             NA         NA           NA       NA        NA
       max_power range_power cv_power
    26        NA          NA       NA

## Вывод

В данной работе мы научились анализировать логи Wi-Fi с использованием
языка R

------------------------------------------------------------------------
