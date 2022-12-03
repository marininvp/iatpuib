# Основы обработки данных с помощью R

## Практическое задание №3

### Цель работы

#### 1. Развить практические навыки использования языка программирования R для обработки данных

#### 2. Закрепить знания базовых типов данных языка R

#### 3. Развить пркатические навыки использования функций обработки данных пакета dplyr – функции select(), filter(), mutate(), arrange(), group\_by()

### Задание

#### Проанализировать встроенный в пакет dplyr набор данных starwars с помощью языка R и ответить на вопросы

### Подготовка

    library(dplyr)

    ## 
    ## Присоединяю пакет: 'dplyr'

    ## Следующие объекты скрыты от 'package:stats':
    ## 
    ##     filter, lag

    ## Следующие объекты скрыты от 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

    starwars

    ## # A tibble: 87 × 14
    ##    name        height  mass hair_…¹ skin_…² eye_c…³ birth…⁴ sex   gender homew…⁵
    ##    <chr>        <int> <dbl> <chr>   <chr>   <chr>     <dbl> <chr> <chr>  <chr>  
    ##  1 Luke Skywa…    172    77 blond   fair    blue       19   male  mascu… Tatooi…
    ##  2 C-3PO          167    75 <NA>    gold    yellow    112   none  mascu… Tatooi…
    ##  3 R2-D2           96    32 <NA>    white,… red        33   none  mascu… Naboo  
    ##  4 Darth Vader    202   136 none    white   yellow     41.9 male  mascu… Tatooi…
    ##  5 Leia Organa    150    49 brown   light   brown      19   fema… femin… Aldera…
    ##  6 Owen Lars      178   120 brown,… light   blue       52   male  mascu… Tatooi…
    ##  7 Beru White…    165    75 brown   light   blue       47   fema… femin… Tatooi…
    ##  8 R5-D4           97    32 <NA>    white,… red        NA   none  mascu… Tatooi…
    ##  9 Biggs Dark…    183    84 black   light   brown      24   male  mascu… Tatooi…
    ## 10 Obi-Wan Ke…    182    77 auburn… fair    blue-g…    57   male  mascu… Stewjon
    ## # … with 77 more rows, 4 more variables: species <chr>, films <list>,
    ## #   vehicles <list>, starships <list>, and abbreviated variable names
    ## #   ¹​hair_color, ²​skin_color, ³​eye_color, ⁴​birth_year, ⁵​homeworld

    starwars <- starwars

### Задание 1

#### 1. Сколько строк в датафрейме?

    starwars %>% nrow()

    ## [1] 87

### Задание 2

#### 2. Сколько столбцов в датафрейме?

    starwars %>% ncol()

    ## [1] 14

### Задание 3

#### 3. Как просмотреть примерный вид датафрейма?

    starwars %>% glimpse()

    ## Rows: 87
    ## Columns: 14
    ## $ name       <chr> "Luke Skywalker", "C-3PO", "R2-D2", "Darth Vader", "Leia Or…
    ## $ height     <int> 172, 167, 96, 202, 150, 178, 165, 97, 183, 182, 188, 180, 2…
    ## $ mass       <dbl> 77.0, 75.0, 32.0, 136.0, 49.0, 120.0, 75.0, 32.0, 84.0, 77.…
    ## $ hair_color <chr> "blond", NA, NA, "none", "brown", "brown, grey", "brown", N…
    ## $ skin_color <chr> "fair", "gold", "white, blue", "white", "light", "light", "…
    ## $ eye_color  <chr> "blue", "yellow", "red", "yellow", "brown", "blue", "blue",…
    ## $ birth_year <dbl> 19.0, 112.0, 33.0, 41.9, 19.0, 52.0, 47.0, NA, 24.0, 57.0, …
    ## $ sex        <chr> "male", "none", "none", "male", "female", "male", "female",…
    ## $ gender     <chr> "masculine", "masculine", "masculine", "masculine", "femini…
    ## $ homeworld  <chr> "Tatooine", "Tatooine", "Naboo", "Tatooine", "Alderaan", "T…
    ## $ species    <chr> "Human", "Droid", "Droid", "Human", "Human", "Human", "Huma…
    ## $ films      <list> <"The Empire Strikes Back", "Revenge of the Sith", "Return…
    ## $ vehicles   <list> <"Snowspeeder", "Imperial Speeder Bike">, <>, <>, <>, "Imp…
    ## $ starships  <list> <"X-wing", "Imperial shuttle">, <>, <>, "TIE Advanced x1",…

### Задание 4

#### 4. Сколько уникальных рас персонажей (species) представлено в данных?

    x <- is.na(starwars$species)
    length(unique(starwars$species[!x]))

    ## [1] 37

### Задание 5

#### 5. Найти самого высокого персонажа.

    starwars[which.max(starwars$height),]$name

    ## [1] "Yarael Poof"

### Задание 6

#### 6. Найти всех персонажей ниже 170

    s <- is.na(starwars$height)
    k <- starwars$height[!s]
    starwars[starwars$height %in% k & starwars$height <170,]$name 

    ##  [1] "C-3PO"                 "R2-D2"                 "Leia Organa"          
    ##  [4] "Beru Whitesun lars"    "R5-D4"                 "Yoda"                 
    ##  [7] "Mon Mothma"            "Wicket Systri Warrick" "Nien Nunb"            
    ## [10] "Watto"                 "Sebulba"               "Shmi Skywalker"       
    ## [13] "Dud Bolt"              "Gasgano"               "Ben Quadinaros"       
    ## [16] "Cordé"                 "Barriss Offee"         "Dormé"                
    ## [19] "Zam Wesell"            "Jocasta Nu"            "Ratts Tyerell"        
    ## [22] "R4-P17"                "Padmé Amidala"

### Задание 7

#### 7. Подсчитать ИМТ (индекс массы тела) для всех персонажей. ИМТ подсчитать по формуле 𝐼 = ℎ𝑚2, где 𝑚 – масса (weight), а ℎ – рост (height).

    imt <- starwars %>% filter(!is.na(mass)) %>% filter(!is.na(height))%>%   group_by(name)  %>% summarise(IMT=mass/(height/100)^2)
    knitr::kable(imt, "pipe")

<table>
<thead>
<tr class="header">
<th style="text-align: left;">name</th>
<th style="text-align: right;">IMT</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Ackbar</td>
<td style="text-align: right;">25.61728</td>
</tr>
<tr class="even">
<td style="text-align: left;">Adi Gallia</td>
<td style="text-align: right;">14.76843</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Anakin Skywalker</td>
<td style="text-align: right;">23.76641</td>
</tr>
<tr class="even">
<td style="text-align: left;">Ayla Secura</td>
<td style="text-align: right;">17.35892</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Barriss Offee</td>
<td style="text-align: right;">18.14487</td>
</tr>
<tr class="even">
<td style="text-align: left;">Ben Quadinaros</td>
<td style="text-align: right;">24.46460</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Beru Whitesun lars</td>
<td style="text-align: right;">27.54821</td>
</tr>
<tr class="even">
<td style="text-align: left;">Biggs Darklighter</td>
<td style="text-align: right;">25.08286</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Boba Fett</td>
<td style="text-align: right;">23.35095</td>
</tr>
<tr class="even">
<td style="text-align: left;">Bossk</td>
<td style="text-align: right;">31.30194</td>
</tr>
<tr class="odd">
<td style="text-align: left;">C-3PO</td>
<td style="text-align: right;">26.89232</td>
</tr>
<tr class="even">
<td style="text-align: left;">Chewbacca</td>
<td style="text-align: right;">21.54509</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Darth Maul</td>
<td style="text-align: right;">26.12245</td>
</tr>
<tr class="even">
<td style="text-align: left;">Darth Vader</td>
<td style="text-align: right;">33.33007</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Dexter Jettster</td>
<td style="text-align: right;">26.01775</td>
</tr>
<tr class="even">
<td style="text-align: left;">Dooku</td>
<td style="text-align: right;">21.47709</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Dud Bolt</td>
<td style="text-align: right;">50.92802</td>
</tr>
<tr class="even">
<td style="text-align: left;">Greedo</td>
<td style="text-align: right;">24.72518</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Gregar Typho</td>
<td style="text-align: right;">24.83565</td>
</tr>
<tr class="even">
<td style="text-align: left;">Grievous</td>
<td style="text-align: right;">34.07922</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Han Solo</td>
<td style="text-align: right;">24.69136</td>
</tr>
<tr class="even">
<td style="text-align: left;">IG-88</td>
<td style="text-align: right;">35.00000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Jabba Desilijic Tiure</td>
<td style="text-align: right;">443.42857</td>
</tr>
<tr class="even">
<td style="text-align: left;">Jango Fett</td>
<td style="text-align: right;">23.58984</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Jar Jar Binks</td>
<td style="text-align: right;">17.18034</td>
</tr>
<tr class="even">
<td style="text-align: left;">Jek Tono Porkins</td>
<td style="text-align: right;">33.95062</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Ki-Adi-Mundi</td>
<td style="text-align: right;">20.91623</td>
</tr>
<tr class="even">
<td style="text-align: left;">Kit Fisto</td>
<td style="text-align: right;">22.64681</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Lama Su</td>
<td style="text-align: right;">16.78076</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lando Calrissian</td>
<td style="text-align: right;">25.21625</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Leia Organa</td>
<td style="text-align: right;">21.77778</td>
</tr>
<tr class="even">
<td style="text-align: left;">Lobot</td>
<td style="text-align: right;">25.79592</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Luke Skywalker</td>
<td style="text-align: right;">26.02758</td>
</tr>
<tr class="even">
<td style="text-align: left;">Luminara Unduli</td>
<td style="text-align: right;">19.44637</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Mace Windu</td>
<td style="text-align: right;">23.76641</td>
</tr>
<tr class="even">
<td style="text-align: left;">Nien Nunb</td>
<td style="text-align: right;">26.56250</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Nute Gunray</td>
<td style="text-align: right;">24.67038</td>
</tr>
<tr class="even">
<td style="text-align: left;">Obi-Wan Kenobi</td>
<td style="text-align: right;">23.24598</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Owen Lars</td>
<td style="text-align: right;">37.87401</td>
</tr>
<tr class="even">
<td style="text-align: left;">Padmé Amidala</td>
<td style="text-align: right;">16.52893</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Palpatine</td>
<td style="text-align: right;">25.95156</td>
</tr>
<tr class="even">
<td style="text-align: left;">Plo Koon</td>
<td style="text-align: right;">22.63468</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Poggle the Lesser</td>
<td style="text-align: right;">23.88844</td>
</tr>
<tr class="even">
<td style="text-align: left;">Qui-Gon Jinn</td>
<td style="text-align: right;">23.89326</td>
</tr>
<tr class="odd">
<td style="text-align: left;">R2-D2</td>
<td style="text-align: right;">34.72222</td>
</tr>
<tr class="even">
<td style="text-align: left;">R5-D4</td>
<td style="text-align: right;">34.00999</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Ratts Tyerell</td>
<td style="text-align: right;">24.03461</td>
</tr>
<tr class="even">
<td style="text-align: left;">Raymus Antilles</td>
<td style="text-align: right;">22.35174</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Roos Tarpals</td>
<td style="text-align: right;">16.34247</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sebulba</td>
<td style="text-align: right;">31.88776</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Shaak Ti</td>
<td style="text-align: right;">17.99015</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sly Moore</td>
<td style="text-align: right;">15.14960</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Tarfful</td>
<td style="text-align: right;">24.83746</td>
</tr>
<tr class="even">
<td style="text-align: left;">Tion Medon</td>
<td style="text-align: right;">18.85192</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Wat Tambor</td>
<td style="text-align: right;">12.88625</td>
</tr>
<tr class="even">
<td style="text-align: left;">Wedge Antilles</td>
<td style="text-align: right;">26.64360</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Wicket Systri Warrick</td>
<td style="text-align: right;">25.82645</td>
</tr>
<tr class="even">
<td style="text-align: left;">Yoda</td>
<td style="text-align: right;">39.02663</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Zam Wesell</td>
<td style="text-align: right;">19.48696</td>
</tr>
</tbody>
</table>

### Задание 8

#### 8. Найти 10 самых “вытянутых” персонажей. “Вытянутость” оценить по отношению массы (mass) к росту (height) персонажей.

    dat <- starwars %>% group_by(name)  %>% summarise(Elongation=mass/height)
    head(arrange(dat,desc(Elongation)),10)

    ## # A tibble: 10 × 2
    ##    name                  Elongation
    ##    <chr>                      <dbl>
    ##  1 Jabba Desilijic Tiure      7.76 
    ##  2 Grievous                   0.736
    ##  3 IG-88                      0.7  
    ##  4 Owen Lars                  0.674
    ##  5 Darth Vader                0.673
    ##  6 Jek Tono Porkins           0.611
    ##  7 Bossk                      0.595
    ##  8 Tarfful                    0.581
    ##  9 Dexter Jettster            0.515
    ## 10 Chewbacca                  0.491

### Задание 9

#### 9. Найти средний возраст персонажей каждой расы вселенной Звездных войн.

    starwars %>% filter(!is.na(birth_year))%>% filter(!is.na(species)) %>% group_by(species)  %>% summarise(age= mean(birth_year))

    ## # A tibble: 15 × 2
    ##    species          age
    ##    <chr>          <dbl>
    ##  1 Cerean          92  
    ##  2 Droid           53.3
    ##  3 Ewok             8  
    ##  4 Gungan          52  
    ##  5 Human           53.4
    ##  6 Hutt           600  
    ##  7 Kel Dor         22  
    ##  8 Mirialan        49  
    ##  9 Mon Calamari    41  
    ## 10 Rodian          44  
    ## 11 Trandoshan      53  
    ## 12 Twi'lek         48  
    ## 13 Wookiee        200  
    ## 14 Yoda's species 896  
    ## 15 Zabrak          54

### Задание 10

#### 10. Найти самый распространенный цвет глаз персонажей вселенной Звездных войн.

    eye <- starwars %>% group_by(eye_color)  %>% summarise(count=n())
    head(arrange(eye,desc(count)),1)

    ## # A tibble: 1 × 2
    ##   eye_color count
    ##   <chr>     <int>
    ## 1 brown        21

### Задание 11

#### 11. Подсчитать среднюю длину имени в каждой расе вселенной Звездных войн.

    sr <- starwars %>% filter(!is.na(species)) %>% group_by(species)  %>% summarise(length=mean(nchar(name)))
    knitr::kable(sr, "pipe")

<table>
<thead>
<tr class="header">
<th style="text-align: left;">species</th>
<th style="text-align: right;">length</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td style="text-align: left;">Aleena</td>
<td style="text-align: right;">13.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Besalisk</td>
<td style="text-align: right;">15.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Cerean</td>
<td style="text-align: right;">12.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Chagrian</td>
<td style="text-align: right;">10.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Clawdite</td>
<td style="text-align: right;">10.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Droid</td>
<td style="text-align: right;">4.833333</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Dug</td>
<td style="text-align: right;">7.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Ewok</td>
<td style="text-align: right;">21.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Geonosian</td>
<td style="text-align: right;">17.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Gungan</td>
<td style="text-align: right;">11.666667</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Human</td>
<td style="text-align: right;">11.285714</td>
</tr>
<tr class="even">
<td style="text-align: left;">Hutt</td>
<td style="text-align: right;">21.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Iktotchi</td>
<td style="text-align: right;">11.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Kaleesh</td>
<td style="text-align: right;">8.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Kaminoan</td>
<td style="text-align: right;">7.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Kel Dor</td>
<td style="text-align: right;">8.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Mirialan</td>
<td style="text-align: right;">14.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Mon Calamari</td>
<td style="text-align: right;">6.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Muun</td>
<td style="text-align: right;">8.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Nautolan</td>
<td style="text-align: right;">9.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Neimodian</td>
<td style="text-align: right;">11.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Pau’an</td>
<td style="text-align: right;">10.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Quermian</td>
<td style="text-align: right;">11.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Rodian</td>
<td style="text-align: right;">6.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Skakoan</td>
<td style="text-align: right;">10.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Sullustan</td>
<td style="text-align: right;">9.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Tholothian</td>
<td style="text-align: right;">10.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Togruta</td>
<td style="text-align: right;">8.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Toong</td>
<td style="text-align: right;">14.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Toydarian</td>
<td style="text-align: right;">5.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Trandoshan</td>
<td style="text-align: right;">5.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Twi’lek</td>
<td style="text-align: right;">11.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Vulptereen</td>
<td style="text-align: right;">8.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Wookiee</td>
<td style="text-align: right;">8.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Xexto</td>
<td style="text-align: right;">7.000000</td>
</tr>
<tr class="even">
<td style="text-align: left;">Yoda’s species</td>
<td style="text-align: right;">4.000000</td>
</tr>
<tr class="odd">
<td style="text-align: left;">Zabrak</td>
<td style="text-align: right;">9.500000</td>
</tr>
</tbody>
</table>
