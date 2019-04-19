Concatenando CSVs num só data frame
================
Thomas Jagoda & Dan Reznik
Abril, 2019

Motivação
---------

Neste projeto mostramos algumas maneiras de combinar arquivos .csv avulsos num só data frame. Analisamos os segintes casos:

-   Quando os arquivos encontram-se avulsos e não compactados
-   Quando os arquivos encontram-se compactados num só "zipfile"

Abaixo almejamos "montar" um só data frame com dados de 27 capitais brasileiras espalhados em 4 csvs no diretório `data` ou num zipfile.

Para fins de simplificação, os arquivos tratados não possuem defeitos, já estão codificados em `UTF-8`, e têm todos o mesmo cabeçalho e número de colunas (o número de linhas é variável entre eles).

### Carrega Pacotes

``` r
library(tidyverse)
library(fs)
library(zip)
```

Concatenação Manual
-------------------

Obtem vetor dos nomes de arquivos CSV filtrados por regexp

``` r
fname_vec <- dir_ls("data",regexp = "capitais\\d{2}\\.csv") %>%
  as.character
fname_vec
#> [1] "data/capitais01.csv" "data/capitais02.csv" "data/capitais03.csv"
#> [4] "data/capitais04.csv"
```

-   Carrega 1o arquivo

``` r
df01 <- read_csv(fname_vec[1])
#> Parsed with column specification:
#> cols(
#>   id = col_double(),
#>   cidade = col_character(),
#>   uf = col_character(),
#>   lat = col_character(),
#>   lon = col_character(),
#>   lat_dec = col_double(),
#>   lon_dec = col_double()
#> )
```

|   id| cidade         | uf  | lat           | lon            |  lat\_dec|  lon\_dec|
|----:|:---------------|:----|:--------------|:---------------|---------:|---------:|
|    1| Aracaju        | SE  | 10º 54' 40" S | 037º 04' 18" O |   -10.911|   -37.072|
|    2| Belém          | PA  | 01º 27' 21" S | 048º 30' 16" O |    -1.456|   -48.504|
|    3| Belo Horizonte | MG  | 19º 55' 15" S | 043º 56' 16" O |   -19.921|   -43.938|
|    4| Boa Vista      | RR  | 02º 49' 11" N | 060º 40' 24" O |     2.820|   -60.673|
|    5| Brasília       | DF  | 15º 46' 47" S | 047º 55' 47" O |   -15.780|   -47.930|

-   Carrega 2o arquivo

``` r
df02 <- read_csv(fname_vec[2])
```

|   id| cidade        | uf  | lat           | lon            |  lat\_dec|  lon\_dec|
|----:|:--------------|:----|:--------------|:---------------|---------:|---------:|
|    6| Campo Grande  | MS  | 20º 26' 34" S | 054º 38' 47" O |   -20.443|   -54.646|
|    7| Cuiabá        | MT  | 15º 35' 46" S | 056º 05' 48" O |   -15.596|   -56.097|
|    8| Curitiba      | PR  | 25º 25' 40" S | 049º 16' 23" O |   -25.428|   -49.273|
|    9| Florianópolis | SC  | 27º 35' 48" S | 048º 32' 57" O |   -27.597|   -48.549|
|   10| Fortaleza     | CE  | 03º 43' 02" S | 038º 32' 35" O |    -3.717|   -38.543|
|   11| Goiânia       | GO  | 16º 40' 43" S | 049º 15' 14" O |   -16.679|   -49.254|
|   12| João Pessoa   | PB  | 07º 06' 54" S | 034º 51' 47" O |    -7.115|   -34.863|

-   Combina 1o e 2o arquivo usando `bind_rows()`

``` r
df01 %>% bind_rows(df02)
#> # A tibble: 12 x 7
#>       id cidade        uf    lat            lon             lat_dec lon_dec
#>    <dbl> <chr>         <chr> <chr>          <chr>             <dbl>   <dbl>
#>  1     1 Aracaju       SE    "10º 54' 40\"… "037º 04' 18\"…  -10.9    -37.1
#>  2     2 Belém         PA    "01º 27' 21\"… "048º 30' 16\"…   -1.46   -48.5
#>  3     3 Belo Horizon… MG    "19º 55' 15\"… "043º 56' 16\"…  -19.9    -43.9
#>  4     4 Boa Vista     RR    "02º 49' 11\"… "060º 40' 24\"…    2.82   -60.7
#>  5     5 Brasília      DF    "15º 46' 47\"… "047º 55' 47\"…  -15.8    -47.9
#>  6     6 Campo Grande  MS    "20º 26' 34\"… "054º 38' 47\"…  -20.4    -54.6
#>  7     7 Cuiabá        MT    "15º 35' 46\"… "056º 05' 48\"…  -15.6    -56.1
#>  8     8 Curitiba      PR    "25º 25' 40\"… "049º 16' 23\"…  -25.4    -49.3
#>  9     9 Florianópolis SC    "27º 35' 48\"… "048º 32' 57\"…  -27.6    -48.5
#> 10    10 Fortaleza     CE    "03º 43' 02\"… "038º 32' 35\"…   -3.72   -38.5
#> 11    11 Goiânia       GO    "16º 40' 43\"… "049º 15' 14\"…  -16.7    -49.3
#> 12    12 João Pessoa   PB    "07º 06' 54\"… "034º 51' 47\"…   -7.12   -34.9
```

-   Carrega 3o arquivo

``` r
df03 <- read_csv(fname_vec[3])
```

|   id| cidade       | uf  | lat           | lon            |  lat\_dec|  lon\_dec|
|----:|:-------------|:----|:--------------|:---------------|---------:|---------:|
|   13| Macapá       | AP  | 00º 02' 20" N | 051º 03' 59" O |     0.039|   -51.066|
|   14| Maceió       | AL  | 09º 39' 57" S | 035º 44' 07" O |    -9.666|   -35.735|
|   15| Manaus       | AM  | 03º 06' 07" S | 060º 01' 30" O |    -3.102|   -60.025|
|   16| Natal        | RN  | 05º 47' 42" S | 035º 12' 34" O |    -5.795|   -35.209|
|   17| Palmas       | TO  | 10º 12' 46" S | 048º 21' 37" O |   -10.213|   -48.360|
|   18| Porto Alegre | RS  | 30º 01' 59" S | 051º 13' 48" O |   -30.033|   -51.230|
|   19| Porto Velho  | RO  | 08º 45' 43" S | 063º 54' 14" O |    -8.762|   -63.904|
|   20| Recife       | PE  | 08º 03' 14" S | 034º 52' 52" O |    -8.054|   -34.881|

-   Combina 1o,2o, e 3o com `bind_rows()`

``` r
df123 <- df01 %>%
  bind_rows(df02) %>%
  bind_rows(df03)
```

|   id| cidade         | uf  | lat           | lon            |  lat\_dec|  lon\_dec|
|----:|:---------------|:----|:--------------|:---------------|---------:|---------:|
|    1| Aracaju        | SE  | 10º 54' 40" S | 037º 04' 18" O |   -10.911|   -37.072|
|    2| Belém          | PA  | 01º 27' 21" S | 048º 30' 16" O |    -1.456|   -48.504|
|    3| Belo Horizonte | MG  | 19º 55' 15" S | 043º 56' 16" O |   -19.921|   -43.938|
|    4| Boa Vista      | RR  | 02º 49' 11" N | 060º 40' 24" O |     2.820|   -60.673|
|    5| Brasília       | DF  | 15º 46' 47" S | 047º 55' 47" O |   -15.780|   -47.930|
|    6| Campo Grande   | MS  | 20º 26' 34" S | 054º 38' 47" O |   -20.443|   -54.646|
|    7| Cuiabá         | MT  | 15º 35' 46" S | 056º 05' 48" O |   -15.596|   -56.097|
|    8| Curitiba       | PR  | 25º 25' 40" S | 049º 16' 23" O |   -25.428|   -49.273|
|    9| Florianópolis  | SC  | 27º 35' 48" S | 048º 32' 57" O |   -27.597|   -48.549|
|   10| Fortaleza      | CE  | 03º 43' 02" S | 038º 32' 35" O |    -3.717|   -38.543|
|   11| Goiânia        | GO  | 16º 40' 43" S | 049º 15' 14" O |   -16.679|   -49.254|
|   12| João Pessoa    | PB  | 07º 06' 54" S | 034º 51' 47" O |    -7.115|   -34.863|
|   13| Macapá         | AP  | 00º 02' 20" N | 051º 03' 59" O |     0.039|   -51.066|
|   14| Maceió         | AL  | 09º 39' 57" S | 035º 44' 07" O |    -9.666|   -35.735|
|   15| Manaus         | AM  | 03º 06' 07" S | 060º 01' 30" O |    -3.102|   -60.025|
|   16| Natal          | RN  | 05º 47' 42" S | 035º 12' 34" O |    -5.795|   -35.209|
|   17| Palmas         | TO  | 10º 12' 46" S | 048º 21' 37" O |   -10.213|   -48.360|
|   18| Porto Alegre   | RS  | 30º 01' 59" S | 051º 13' 48" O |   -30.033|   -51.230|
|   19| Porto Velho    | RO  | 08º 45' 43" S | 063º 54' 14" O |    -8.762|   -63.904|
|   20| Recife         | PE  | 08º 03' 14" S | 034º 52' 52" O |    -8.054|   -34.881|

-   Maneira alternativa de combiná-los

``` r
df123a <- bind_rows(df01,df02,df03)
```

|   id| cidade         | uf  | lat           | lon            |  lat\_dec|  lon\_dec|
|----:|:---------------|:----|:--------------|:---------------|---------:|---------:|
|    1| Aracaju        | SE  | 10º 54' 40" S | 037º 04' 18" O |   -10.911|   -37.072|
|    2| Belém          | PA  | 01º 27' 21" S | 048º 30' 16" O |    -1.456|   -48.504|
|    3| Belo Horizonte | MG  | 19º 55' 15" S | 043º 56' 16" O |   -19.921|   -43.938|
|    4| Boa Vista      | RR  | 02º 49' 11" N | 060º 40' 24" O |     2.820|   -60.673|
|    5| Brasília       | DF  | 15º 46' 47" S | 047º 55' 47" O |   -15.780|   -47.930|
|    6| Campo Grande   | MS  | 20º 26' 34" S | 054º 38' 47" O |   -20.443|   -54.646|
|    7| Cuiabá         | MT  | 15º 35' 46" S | 056º 05' 48" O |   -15.596|   -56.097|
|    8| Curitiba       | PR  | 25º 25' 40" S | 049º 16' 23" O |   -25.428|   -49.273|
|    9| Florianópolis  | SC  | 27º 35' 48" S | 048º 32' 57" O |   -27.597|   -48.549|
|   10| Fortaleza      | CE  | 03º 43' 02" S | 038º 32' 35" O |    -3.717|   -38.543|
|   11| Goiânia        | GO  | 16º 40' 43" S | 049º 15' 14" O |   -16.679|   -49.254|
|   12| João Pessoa    | PB  | 07º 06' 54" S | 034º 51' 47" O |    -7.115|   -34.863|
|   13| Macapá         | AP  | 00º 02' 20" N | 051º 03' 59" O |     0.039|   -51.066|
|   14| Maceió         | AL  | 09º 39' 57" S | 035º 44' 07" O |    -9.666|   -35.735|
|   15| Manaus         | AM  | 03º 06' 07" S | 060º 01' 30" O |    -3.102|   -60.025|
|   16| Natal          | RN  | 05º 47' 42" S | 035º 12' 34" O |    -5.795|   -35.209|
|   17| Palmas         | TO  | 10º 12' 46" S | 048º 21' 37" O |   -10.213|   -48.360|
|   18| Porto Alegre   | RS  | 30º 01' 59" S | 051º 13' 48" O |   -30.033|   -51.230|
|   19| Porto Velho    | RO  | 08º 45' 43" S | 063º 54' 14" O |    -8.762|   -63.904|
|   20| Recife         | PE  | 08º 03' 14" S | 034º 52' 52" O |    -8.054|   -34.881|

Concatenação Automatizada
-------------------------

Combina com `purrr::map_dfr()`, note que o data frame de saída possui 27 linhas

``` r
df_all <- fname_vec %>%
  map_dfr(read_csv)
```

|   id| cidade         | uf  | lat           | lon            |  lat\_dec|  lon\_dec|
|----:|:---------------|:----|:--------------|:---------------|---------:|---------:|
|    1| Aracaju        | SE  | 10º 54' 40" S | 037º 04' 18" O |   -10.911|   -37.072|
|    2| Belém          | PA  | 01º 27' 21" S | 048º 30' 16" O |    -1.456|   -48.504|
|    3| Belo Horizonte | MG  | 19º 55' 15" S | 043º 56' 16" O |   -19.921|   -43.938|
|    4| Boa Vista      | RR  | 02º 49' 11" N | 060º 40' 24" O |     2.820|   -60.673|
|    5| Brasília       | DF  | 15º 46' 47" S | 047º 55' 47" O |   -15.780|   -47.930|
|    6| Campo Grande   | MS  | 20º 26' 34" S | 054º 38' 47" O |   -20.443|   -54.646|
|    7| Cuiabá         | MT  | 15º 35' 46" S | 056º 05' 48" O |   -15.596|   -56.097|
|    8| Curitiba       | PR  | 25º 25' 40" S | 049º 16' 23" O |   -25.428|   -49.273|
|    9| Florianópolis  | SC  | 27º 35' 48" S | 048º 32' 57" O |   -27.597|   -48.549|
|   10| Fortaleza      | CE  | 03º 43' 02" S | 038º 32' 35" O |    -3.717|   -38.543|
|   11| Goiânia        | GO  | 16º 40' 43" S | 049º 15' 14" O |   -16.679|   -49.254|
|   12| João Pessoa    | PB  | 07º 06' 54" S | 034º 51' 47" O |    -7.115|   -34.863|
|   13| Macapá         | AP  | 00º 02' 20" N | 051º 03' 59" O |     0.039|   -51.066|
|   14| Maceió         | AL  | 09º 39' 57" S | 035º 44' 07" O |    -9.666|   -35.735|
|   15| Manaus         | AM  | 03º 06' 07" S | 060º 01' 30" O |    -3.102|   -60.025|
|   16| Natal          | RN  | 05º 47' 42" S | 035º 12' 34" O |    -5.795|   -35.209|
|   17| Palmas         | TO  | 10º 12' 46" S | 048º 21' 37" O |   -10.213|   -48.360|
|   18| Porto Alegre   | RS  | 30º 01' 59" S | 051º 13' 48" O |   -30.033|   -51.230|
|   19| Porto Velho    | RO  | 08º 45' 43" S | 063º 54' 14" O |    -8.762|   -63.904|
|   20| Recife         | PE  | 08º 03' 14" S | 034º 52' 52" O |    -8.054|   -34.881|
|   21| Rio Branco     | AC  | 09º 58' 29" S | 067º 48' 36" O |    -9.975|   -67.810|
|   22| Rio de Janeiro | RJ  | 22º 54' 10" S | 043º 12' 27" O |   -22.903|   -43.208|
|   23| Salvador       | BA  | 12º 58' 16" S | 038º 30' 39" O |   -12.971|   -38.511|
|   24| São Luís       | MA  | 02º 31' 47" S | 044º 18' 10" O |    -2.530|   -44.303|
|   25| São Paulo      | SP  | 23º 32' 51" S | 046º 38' 10" O |   -23.548|   -46.636|
|   26| Teresina       | PI  | 05º 05' 21" S | 042º 48' 07" O |    -5.089|   -42.802|
|   27| Vitória        | ES  | 20º 19' 10" S | 040º 20' 16" O |   -20.319|   -40.338|

### Concatenação a partir de um zip

Lista arquivos no diretório `data` com a extensão "zip"

``` r
zipfile <- dir_ls("data",regexp="\\.zip$") %>% as.character
zipfile
#> [1] "data/capitais.zip"
```

Lista arquivos dentro do arquivo encontrado

``` r
fnames_zip <- zip_list(zipfile)
```

Função auxiliar, a ser iterada abaixo, que executará as seguintes operações:

1.  extrai um arquivo do zipfile no diretorio corrente
2.  lê data frame com `read_csv()`
3.  deleta o arquivo
4.  retorna o dataframe lido para uso por `map_dfr()`

``` r
unzip_read_delete <- function(fname_csv,zipfile) {
  unzip(zipfile,fname_csv)
  df <- read_csv(fname_csv)
  file_delete(fname_csv)
  df # importante retornar um tibble q sera usado por map_dfr()
}
```

Itera sobre arquivos no zip, concatenando-os com `map_dfr()`

``` r
df_all_map <- fnames_zip$filename %>%
  map_dfr(unzip_read_delete,zipfile)
# map_dfr(~unzip_read_delete(.x,zipfile))
```

|   id| cidade         | uf  | lat           | lon            |  lat\_dec|  lon\_dec|
|----:|:---------------|:----|:--------------|:---------------|---------:|---------:|
|    1| Aracaju        | SE  | 10º 54' 40" S | 037º 04' 18" O |   -10.911|   -37.072|
|    2| Belém          | PA  | 01º 27' 21" S | 048º 30' 16" O |    -1.456|   -48.504|
|    3| Belo Horizonte | MG  | 19º 55' 15" S | 043º 56' 16" O |   -19.921|   -43.938|
|    4| Boa Vista      | RR  | 02º 49' 11" N | 060º 40' 24" O |     2.820|   -60.673|
|    5| Brasília       | DF  | 15º 46' 47" S | 047º 55' 47" O |   -15.780|   -47.930|
|    6| Campo Grande   | MS  | 20º 26' 34" S | 054º 38' 47" O |   -20.443|   -54.646|
|    7| Cuiabá         | MT  | 15º 35' 46" S | 056º 05' 48" O |   -15.596|   -56.097|
|    8| Curitiba       | PR  | 25º 25' 40" S | 049º 16' 23" O |   -25.428|   -49.273|
|    9| Florianópolis  | SC  | 27º 35' 48" S | 048º 32' 57" O |   -27.597|   -48.549|
|   10| Fortaleza      | CE  | 03º 43' 02" S | 038º 32' 35" O |    -3.717|   -38.543|
|   11| Goiânia        | GO  | 16º 40' 43" S | 049º 15' 14" O |   -16.679|   -49.254|
|   12| João Pessoa    | PB  | 07º 06' 54" S | 034º 51' 47" O |    -7.115|   -34.863|
|   13| Macapá         | AP  | 00º 02' 20" N | 051º 03' 59" O |     0.039|   -51.066|
|   14| Maceió         | AL  | 09º 39' 57" S | 035º 44' 07" O |    -9.666|   -35.735|
|   15| Manaus         | AM  | 03º 06' 07" S | 060º 01' 30" O |    -3.102|   -60.025|
|   16| Natal          | RN  | 05º 47' 42" S | 035º 12' 34" O |    -5.795|   -35.209|
|   17| Palmas         | TO  | 10º 12' 46" S | 048º 21' 37" O |   -10.213|   -48.360|
|   18| Porto Alegre   | RS  | 30º 01' 59" S | 051º 13' 48" O |   -30.033|   -51.230|
|   19| Porto Velho    | RO  | 08º 45' 43" S | 063º 54' 14" O |    -8.762|   -63.904|
|   20| Recife         | PE  | 08º 03' 14" S | 034º 52' 52" O |    -8.054|   -34.881|
|   21| Rio Branco     | AC  | 09º 58' 29" S | 067º 48' 36" O |    -9.975|   -67.810|
|   22| Rio de Janeiro | RJ  | 22º 54' 10" S | 043º 12' 27" O |   -22.903|   -43.208|
|   23| Salvador       | BA  | 12º 58' 16" S | 038º 30' 39" O |   -12.971|   -38.511|
|   24| São Luís       | MA  | 02º 31' 47" S | 044º 18' 10" O |    -2.530|   -44.303|
|   25| São Paulo      | SP  | 23º 32' 51" S | 046º 38' 10" O |   -23.548|   -46.636|
|   26| Teresina       | PI  | 05º 05' 21" S | 042º 48' 07" O |    -5.089|   -42.802|
|   27| Vitória        | ES  | 20º 19' 10" S | 040º 20' 16" O |   -20.319|   -40.338|
