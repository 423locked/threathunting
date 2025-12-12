# Практическая работа №2. Основы обработки данных с помощью R и Dplyr
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

## Задание

Проанализировать встроенный в пакет dplyr набор данных starwars с
помощью языка R и ответить на вопросы

## Ход работы

1.  Проанализировать встроенный в пакет dplyr набор данных starwars с
    помощью языка R и ответить на вопросы:
    -   Сколько строк в датафрейме?
    -   Сколько столбцов в датафрейме?
    -   Как просмотреть примерный вид датафрейма?
    -   Сколько уникальных рас персонажей (species) представлено в
        данных?
    -   Найти самого высокого персонажа.
    -   Найти всех персонажей ниже 170
    -   Подсчитать ИМТ (индекс массы тела) для всех персонажей.
    -   Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по
        отношению массы (mass) к росту (height) персонажей
    -   Найти средний возраст персонажей каждой расы вселенной Звездных
        войн
    -   Найти самый распространенный цвет глаз персонажей вселенной
        Звездных войн.
    -   Подсчитать среднюю длину имени в каждой расе вселенной Звездных
        войн.
2.  Оценить результаты и сделать вывод

## Шаги:

### Шаг 1

#### 1. Загрузка и предварительный анализ данных

    > library(dplyr)

    Attaching package: ‘dplyr’

    The following objects are masked from ‘package:stats’:

        filter, lag

    The following objects are masked from ‘package:base’:

        intersect, setdiff, setequal, union
    > data(starwars)
    > 

#### Вопрос 1: Сколько строк в датафрейме?

    > nrow(starwars)
    [1] 87

#### Вопрос 2: Сколько столбцов в датафрейме?

    > ncol(starwars)
    [1] 14

#### Вопрос 3: Как просмотреть примерный вид датафрейма?

    > glimpse(starwars)
    > glimpse(starwars)
    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Organa", "Owen Lars", "Beru Whitesun Lars", "R5-D4", "Biggs Darklighter", "…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 228, 180, 173, 175, 170, 180, 66, 170, 183, 200, 190, 177, 175, 180, 150, …
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.0, 84.0, NA, 112.0, 80.0, 74.0, 1358.0, 77.0, 110.0, 17.0, 75.0, 78.2, 14…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", NA, "black", "auburn, white", "blond", "auburn, grey", "brown", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "light", "white, red", "light", "fair", "fair", "fair", "unknown", "fair",…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue", "red", "brown", "blue-gray", "blue", "blue", "blue", "brown", "black", "…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, 41.9, 64.0, 200.0, 29.0, 44.0, 600.0, 21.0, NA, 896.0, 82.0, 31.5, 15.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female", "none", "male", "male", "male", "male", "male", "male", "male", "hermaph…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "feminine", "masculine", "feminine", "masculine", "masculine", "masculine", "mas…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "Tatooine", "Tatooine", "Tatooine", "Tatooine", "Stewjon", "Tatooine", "Eri…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Human", "Droid", "Human", "Human", "Human", "Human", "Wookiee", "Human", "Rod…
    $ films      <list> <"A New Hope", "The Empire Strikes Back", "Return of the Jedi", "Revenge of the Sith", "The Force Awakens">, <"A New Hope", "The Em…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imperial Speeder Bike", <>, <>, <>, <>, "Tribubble bongo", <"Zephyr-G swoop …
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1", <>, <>, <>, <>, "X-wing", <"Jedi starfighter", "Trade Federation cruiser…
    > 

#### Вопрос 4: Сколько уникальных рас персонажей (species) представлено в данных?

    > starwars %>% distinct(species) %>% nrow()
    [1] 38
    > 
    > 

#### Вопрос 5: Найти самого высокого персонажа

    > starwars %>%
    +     filter(height == max(height,na.rm = TRUE)) %>%
    +     pull(name)
    [1] "Yarael Poof"

#### Вопрос 6: Найти всех персонажей ниже 170

    > starwars %>%
    +     filter(height < 170) %>%
    +     select(name, height)
    # A tibble: 22 × 2
       name                  height
       <chr>                  <int>
     1 C-3PO                    167
     2 R2-D2                     96
     3 Leia Organa              150
     4 Beru Whitesun Lars       165
     5 R5-D4                     97
     6 Yoda                      66
     7 Mon Mothma               150
     8 Wicket Systri Warrick     88
     9 Nien Nunb                160
    10 Watto                    137
    # ℹ 12 more rows
    # ℹ Use `print(n = ...)` to see more rows
    > 
    > 

#### Вопрос 7: Подсчитать ИМТ (индекс массы тела) для всех персонажей.

    > starwars %>%
    +     mutate(height_m = height / 100,bmi = mass / (height_m)^2) %>%
    +     select(name, mass, height, bmi)
    # A tibble: 87 × 4
       name                mass height   bmi
       <chr>              <dbl>  <int> <dbl>
     1 Luke Skywalker        77    172  26.0
     2 C-3PO                 75    167  26.9
     3 R2-D2                 32     96  34.7
     4 Darth Vader          136    202  33.3
     5 Leia Organa           49    150  21.8
     6 Owen Lars            120    178  37.9
     7 Beru Whitesun Lars    75    165  27.5
     8 R5-D4                 32     97  34.0
     9 Biggs Darklighter     84    183  25.1
    10 Obi-Wan Kenobi        77    182  23.2
    # ℹ 77 more rows
    # ℹ Use `print(n = ...)` to see more rows
    > 
    > 

#### Вопрос 8: Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по отношению массы (mass) к росту (height) персонажей.

    > starwars %>% 
    +     mutate(stretch_ratio = mass / height) %>%
    +     filter(!is.na(stretch_ratio)) %>%
    +     arrange(desc(stretch_ratio)) %>%
    +     head(10) %>%
    +     select(name, mass, height, stretch_ratio)
    # A tibble: 10 × 4
       name                   mass height stretch_ratio
       <chr>                 <dbl>  <int>         <dbl>
     1 Jabba Desilijic Tiure  1358    175         7.76 
     2 Grievous                159    216         0.736
     3 IG-88                   140    200         0.7  
     4 Owen Lars               120    178         0.674
     5 Darth Vader             136    202         0.673
     6 Jek Tono Porkins        110    180         0.611
     7 Bossk                   113    190         0.595
     8 Tarfful                 136    234         0.581
     9 Dexter Jettster         102    198         0.515
    10 Chewbacca               112    228         0.491
    > 
    > 

#### Вопрос 9: Найти средний возраст персонажей каждой расы вселенной Звездных войн.

    > starwars %>% 
    +     filter(!is.na(species), !is.na(birth_year)) %>%
    +     group_by(species) %>%
    +     summarise(
    +         count = n(),
    +         avg_birth_year = mean(birth_year, na.rm = TRUE)
    +     ) %>% 
    +     arrange(desc(avg_birth_year))
    # A tibble: 15 × 3
       species        count avg_birth_year
       <chr>          <int>          <dbl>
     1 Yoda's species     1          896  
     2 Hutt               1          600  
     3 Wookiee            1          200  
     4 Cerean             1           92  
     5 Zabrak             1           54  
     6 Human             26           53.7
     7 Droid              3           53.3
     8 Trandoshan         1           53  
     9 Gungan             1           52  
    10 Mirialan           2           49  
    11 Twi'lek            1           48  
    12 Rodian             1           44  
    13 Mon Calamari       1           41  
    14 Kel Dor            1           22  
    15 Ewok               1            8  
    > 

#### Вопрос 10: Найти самый распространенный цвет глаз персонажей вселенной Звездных войн.

    > starwars %>%
    +     group_by(eye_color) %>%
    +     summarise(n = n(), .groups = "drop") %>%
    +     arrange(desc(n)) %>%
    +     slice(1)
    # A tibble: 1 × 2
      eye_color     n
      <chr>     <int>
    1 brown        21
    > 

#### Вопрос 11: Подсчитать среднюю длину имени в каждой расе вселенной Звездных войн.

    > starwars %>%
    +     filter(!is.na(species), !is.na(name)) %>%
    +     group_by(species) %>%
    +     summarise(
    +         count = n(),
    +         avg_length = mean(nchar(name)),
    +         .groups = "drop"
    +     )
    # A tibble: 37 × 3
       species   count avg_length
       <chr>     <int>      <dbl>
     1 Aleena        1      12   
     2 Besalisk      1      15   
     3 Cerean        1      12   
     4 Chagrian      1      10   
     5 Clawdite      1      10   
     6 Droid         6       4.83
     7 Dug           1       7   
     8 Ewok          1      21   
     9 Geonosian     1      17   
    10 Gungan        3      11.7 
    # ℹ 27 more rows
    # ℹ Use `print(n = ...)` to see more rows
    > 

### Шаг 2

#### Оценка результатов

В ходе выполнения работы были успешно применены функции пакета dplyr для
анализа набора данных starwars (`select`, `filter`, `mutate`, `arrange`,
`group_by`, `summarise`).

#### Вывод

В результате выполнения практической работы удалось получить навыки
использования языка `R` и пакета `dplyr` для обработки и анализа данных.
Были успешно применены ключевые функции dplyr для решения конкретных
аналитических задач на реальном наборе данных. —
