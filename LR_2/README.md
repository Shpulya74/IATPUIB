# Лабораторная работа №2
Поспелова Ульяна БИСО-03-20

## Цель работы

1.  Развить практические навыки использования языка программирования R
    для обработки данных.

2.  Закрепить знания базовых типов данных языка R.

3.  Развить пркатические навыки использования функций обработки данных
    пакета dplyr – функции select(), filter(), mutate(), arrange(),
    group_by().

## Задание

Проанализировать встроенный в пакет dplyr набор данных starwars с
помощью языка R и ответить на вопросы:

1\. Сколько строк в датафрейме?

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
starwars %>% nrow()
```

    [1] 87

2\. Сколько столбцов в датафрейме?

``` r
starwars %>% ncol()
```

    [1] 14

3\. Как просмотреть примерный вид датафрейма?

``` r
starwars %>% glimpse()
```

    Rows: 87
    Columns: 14
    $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    $ films      <list> <"The Empire Strikes Back", "Revenge of the Sith", "Return…
    $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

4\. Сколько уникальных рас персонажей (species) представлено в данных?

``` r
length(unique(starwars$species))
```

    [1] 38

5\. Найти самого высокого персонажа.

``` r
starwars %>% slice_max(height) %>% select(name,height)
```

    # A tibble: 1 × 2
      name        height
      <chr>        <int>
    1 Yarael Poof    264

6\. Найти всех персонажей ниже 170

``` r
starwars %>% filter(height<170) %>% select(name,height)
```

    # A tibble: 23 × 2
       name                  height
       <chr>                  <int>
     1 C-3PO                    167
     2 R2-D2                     96
     3 Leia Organa              150
     4 Beru Whitesun lars       165
     5 R5-D4                     97
     6 Yoda                      66
     7 Mon Mothma               150
     8 Wicket Systri Warrick     88
     9 Nien Nunb                160
    10 Watto                    137
    # ℹ 13 more rows

7\. Подсчитать ИМТ (индекс массы тела) для всех персонажей. ИМТ
подсчитать по формуле 𝐼 = 𝑚/ℎ2 , где 𝑚 – масса (weight), а ℎ – рост
(height).

``` r
starwars %>% mutate(IMT = mass/(height/100)**2) %>% select(name,IMT)
```

    # A tibble: 87 × 2
       name                 IMT
       <chr>              <dbl>
     1 Luke Skywalker      26.0
     2 C-3PO               26.9
     3 R2-D2               34.7
     4 Darth Vader         33.3
     5 Leia Organa         21.8
     6 Owen Lars           37.9
     7 Beru Whitesun lars  27.5
     8 R5-D4               34.0
     9 Biggs Darklighter   25.1
    10 Obi-Wan Kenobi      23.2
    # ℹ 77 more rows

8\. Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по
отношению массы (mass) к росту (height) персонажей.

``` r
starwars %>% mutate(vyt = mass/(height/100)) %>% arrange(vyt) %>% select(name,vyt) %>% top_n(10)
```

    Selecting by vyt

    # A tibble: 10 × 2
       name                    vyt
       <chr>                 <dbl>
     1 Chewbacca              49.1
     2 Dexter Jettster        51.5
     3 Tarfful                58.1
     4 Bossk                  59.5
     5 Jek Tono Porkins       61.1
     6 Darth Vader            67.3
     7 Owen Lars              67.4
     8 IG-88                  70  
     9 Grievous               73.6
    10 Jabba Desilijic Tiure 776  

9\. Найти средний возраст персонажей каждой расы вселенной Звездных
войн.

``` r
starwars %>% group_by(species) %>% summarise(mid_age = mean(birth_year, na.rm=TRUE))
```

    # A tibble: 38 × 2
       species   mid_age
       <chr>       <dbl>
     1 Aleena      NaN  
     2 Besalisk    NaN  
     3 Cerean       92  
     4 Chagrian    NaN  
     5 Clawdite    NaN  
     6 Droid        53.3
     7 Dug         NaN  
     8 Ewok          8  
     9 Geonosian   NaN  
    10 Gungan       52  
    # ℹ 28 more rows

10\. Найти самый распространенный цвет глаз персонажей вселенной
Звездных войн.

``` r
starwars %>% group_by(eye_color) %>% summarise(col = n()) %>% slice_max(col)
```

    # A tibble: 1 × 2
      eye_color   col
      <chr>     <int>
    1 brown        21

11\. Подсчитать среднюю длину имени в каждой расе вселенной Звездных
войн.

``` r
starwars %>% group_by(species) %>% summarise(mid_len = mean(length(name)))
```

    # A tibble: 38 × 2
       species   mid_len
       <chr>       <dbl>
     1 Aleena          1
     2 Besalisk        1
     3 Cerean          1
     4 Chagrian        1
     5 Clawdite        1
     6 Droid           6
     7 Dug             1
     8 Ewok            1
     9 Geonosian       1
    10 Gungan          3
    # ℹ 28 more rows

12\. Оформить отчет в соответствии с шаблоном
