Datos temporales en R
================
Héctor Villalobos

## Introducción

Hablamos de datos temporales cuando las variables que se recolectan
tienen asociada una fecha y posiblemente una hora. El uso correcto de
estos datos depende de definir con toda claridad el formato en que esta
información se almacena. Por ejemplo, en los archivos (ascii o Excel) se
puede guardar el año, mes y día en columnas separadas, o combinados en
una misma columna como una cadena de texto o bien en un formato
especializado de fecha (en el caso de Excel).

Usualmente se usan solo números, por lo que tendríamos algo así como
“08-10-2025” (dd-mm-aaaa), aunque en algunos casos podría escribirse
como “2025/Oct/08” (aaaa/mmm/dd). El tiempo podría estar en una columna
aparte o combinado con la fecha: “2025-10-08 15:30:58” (aaaa-mm-dd
hh:mm:ss).

Lo importante en todos los casos es la consistencia, y sobretodo no
invertir el orden del mes y el día ni escribir indistintamente “Sep”,
“sep” o “Septiembre” . Es sumamente importante revisar esto antes de
importar nuestros datos en R.

<img src="fechas.jpg" data-fig-align="center" width="266" />

Al importar en R desde archivos ascii con la función `read.table()`, la
clase de la variable con la fecha será `"character"`, lo cual
dificultará su uso en gráficos o para realizar operaciones, por lo que
debemos convertirlos a una clase apropiada para fecha o fecha-tiempo. En
cambio, al importar archivos Excel con la función `read_excel()` del
paquete **readxl** usualmente ya pasan como clase `POSIXct`, si el
formato no es ambiguo. A continuación se describen algunas de estas
clases especializadas.

## Clase `"Date"`

Esta es tal vez la clase más sencilla cuando solo tenemos fechas.

``` r
fechas <- paste("2025", c("7", "10", "11"), c("13", "18", "25"), sep ="-")
fechas
```

    [1] "2025-7-13"  "2025-10-18" "2025-11-25"

En el ejemplo, el vector `fechas` es de clase `"character"`.

``` r
class(fechas)
```

    [1] "character"

La conversión a la clase `"Date"` se logra con la función `as.Date().`

``` r
fechas <- as.Date(fechas)
fechas
```

    [1] "2025-07-13" "2025-10-18" "2025-11-25"

``` r
class(fechas)
```

    [1] "Date"

Esta clase representa el número de días desde el 1 de enero de 1970.

``` r
unclass(fechas)
```

    [1] 20282 20379 20417

Entonces, considerando que al 1 de febrero de 1970 transcurrieron 31
días desde el origen mencionado antes,

``` r
 as.Date(31)
```

    [1] "1970-02-01"

Con la función `Sys.time()` podemos obtener la fecha-hora actual de
acuerdo a la configuración local de nuestra computadora.

``` r
ahora <- Sys.time()
ahora
```

    [1] "2025-09-30 17:25:51 MST"

Al aplicar la función `as.Date()` la hora se descarta.

``` r
as.Date(ahora)
```

    [1] "2025-10-01"

## La clase `"POSIXlt"`

Las clases más utilizadas en R que permiten guardar la fecha, hora y la
zona de tiempo son **`"POSIXlt"`** y **`"POSIXct"`**. La primera
almacena esta información como una lista de vectores `sec`, `min`,
`hour` para el tiempo; `mday`, `mon` y `year` para la fecha; `wday` y
`yday` para el día de la semana y el día del año,
respectivamente;`isdst`, es una bandera para el horario de verano;
`zone` es una cadena de texto para la zona de tiempo; y `gmtoff` sería
el *offset* en segundos del horario GMT. La función `strptime()` nos
permite convertir una cadena de texto a la clase `"POSIXlt"`.

``` r
dt <- strptime("2025-10-08 07:12:30", format = "%Y-%m-%d %H:%M:%S", tz = "UTC")
dt
```

    [1] "2025-10-08 07:12:30 UTC"

``` r
class(dt)
```

    [1] "POSIXlt" "POSIXt" 

Pare ver los vectores mencionados antes, usamos nuevamente la función
`unclass()`.

``` r
unclass(dt)
```

    $sec
    [1] 30

    $min
    [1] 12

    $hour
    [1] 7

    $mday
    [1] 8

    $mon
    [1] 9

    $year
    [1] 125

    $wday
    [1] 3

    $yday
    [1] 280

    $isdst
    [1] 0

    $zone
    [1] "UTC"

    $gmtoff
    [1] 0

    attr(,"tzone")
    [1] "UTC"
    attr(,"balanced")
    [1] TRUE

También podemos usar la función `as.POSIXlt()` para convertir una cadena
de texto a la clase `"POSIXlt"`. En este caso, si no lo indicamos la
zona de tiempo se ajustará de acuerdo a nuestra configuración local.

``` r
psx.lt <- as.POSIXlt("2025-10-08 07:12:30")
psx.lt
```

    [1] "2025-10-08 07:12:30 MST"

``` r
class(psx.lt)
```

    [1] "POSIXlt" "POSIXt" 

## La clase `"POSIXct"`

Esta clase representa el número de segundos desde el primero de enero de
1970 (en UTC) como un vector numérico.

``` r
psx.ct <- as.POSIXct("2024-10-17 07:12:30", tz = "UTC")
psx.ct
```

    [1] "2024-10-17 07:12:30 UTC"

``` r
class(psx.ct)
```

    [1] "POSIXct" "POSIXt" 

Aplicando la función `unclass()` podemos ver el número de segundos
transcurridos desde el origen mencionado antes.

``` r
unclass(psx.ct)
```

    [1] 1729149150
    attr(,"tzone")
    [1] "UTC"

Esto lo podemos comprobar fácilmente.

``` r
# definir origen
orig <- as.POSIXct("1970-01-01 00:00:00", format = "%Y-%m-%d %H:%M:%S", tz = "UTC") 

# calcular la diferencia entre psx.ct y el origen
difftime(psx.ct, orig, units = "s")
```

    Time difference of 1729149150 secs

De acuerdo con la ayuda de R, la clase `"POSIXct"` es más conveniente
para incluirse en *data frames*, mientras que `"POSIXlt"`es una forma
más fácil de leer para el humano.

## Manipulación de datos temporales: temperatura potencial del mar diaria

Para ilustrar la manipulación de datos temporales utilizaremos
información del producto *Global Ocean Physics Reanalysis*
(<https://doi.org/10.48670/moi-00021>) de Copernicus
(<https://marine.copernicus.eu/>).

El archivo netCDF proporcionado contiene datos diarios de temperatura
potencial para una región del Golfo de California, México. El periodo
abarca del primero de enero de 2019 al 31 de diciembre de 2020. En este
archivo la información de la fecha podemos verla con el siguiente código

``` r
library(ncdf4)
ncf <- nc_open("./datos/cmems_mod_glo_phy_my_0.083deg_P1D-m_1728505215428.nc")
time <- ncf$dim$time$vals
head(time)
```

    [1] 1546300800 1546387200 1546473600 1546560000 1546646400 1546732800

Inspeccionando un poco el archivo (tecleando ncf en la consola) podemos
notar que estos números son: “units: seconds since 1970-01-01 00:00:00”,
por lo que la conversión apropiada sería

``` r
time <- as.POSIXct(time, tz = "UTC", format = "%Y-%m-%d", 
                   origin = "1970-01-01 00:00:00")
head(time)
```

    [1] "2019-01-01 UTC" "2019-01-02 UTC" "2019-01-03 UTC" "2019-01-04 UTC"
    [5] "2019-01-05 UTC" "2019-01-06 UTC"

El proceso anterior se encuentra codificado en la función `read.cmems()`
del paquete **satin**, que además prevee otros casos en los que el
origen o las unidades (horas, días) pudieran ser diferentes.

``` r
# cargar paquete devtools
#library(devtools)

# instalar 'satin' desde github
#install_github("hvillalo/satin")

# cargar paquete satin
library(satin)
```

Importar el archivo netCDF anterior con la función `read.cmems()`.

``` r
thetao <- read.cmems("./datos/cmems_mod_glo_phy_my_0.083deg_P1D-m_1728505215428.nc")
```

Ahora inspeccionamos el archivo importado, comenzando por la clase del
objeto `"thetao"`.

``` r
class(thetao)
```

    [1] "satin"
    attr(,"package")
    [1] "satin"

y su contenido…

``` r
thetao
```

    Object of class satin

     Title: thetao 
     Long name: Temperature 
     Name: thetao 
     Units: degrees_C 
     Temporal range: daily 
     Spatial resolution: 9.2 km 

    Data dimensions:
     61 60 731 2 

    Data ranges:
              lon lat   thetao     period    depth
    min -112.0000  22 13.78762 2019-01-01 0.494025
    max -107.0833  27 32.84582 2020-12-31 1.541375

Podemos ver que tenemos datos diarios de temperatura en °C, con una
resolución espacial de 9.2 km. En total son 731 días , del 2019-01-01 al
2020-12-31 y a 2 niveles de profundidad diferentes, 0.49 y 1.54 m. En
este objeto de clase S4 podemos extraer sus diferentes componentes
(“slots”) usando “@”:

``` r
head(thetao@period$tmStart); tail(thetao@period$tmStart)
```

    [1] "2019-01-01 UTC" "2019-01-02 UTC" "2019-01-03 UTC" "2019-01-04 UTC"
    [5] "2019-01-05 UTC" "2019-01-06 UTC"

    [1] "2020-12-26 UTC" "2020-12-27 UTC" "2020-12-28 UTC" "2020-12-29 UTC"
    [5] "2020-12-30 UTC" "2020-12-31 UTC"

Haremos un mapa para elegir un punto y extraer los valores de
temperatura en el nivel más superficial

``` r
plot(thetao)
```

![](Datos_Temporales_en_R_files/figure-commonmark/unnamed-chunk-21-1.png)

Tomemos por ejemplo un pixel a los 26° de lat N y 110° de lon W

``` r
pt <- data.frame(x = -110, y = 26)
sst <- extractPts(thetao, points = pt)
dim(sst)
```

    [1]    1 1468

En `sst` están los valores de temperatura potencial para el punto
seleccionado, en los 731 días y para los 5 niveles de profundidad 731
$\times$ 5 = 3655. Las primeras seis columnas adicionales en `sst`
contienen el id del punto o puntos elegidos, las coordenadas de
latitud-longitud elegidas, las coordenadas del pixel más cercano donde
hay datos y la distancia entre el punto elegido y el dato devuelto, solo
como un control

``` r
sst[ , 1:10]
```

      id    x  y d  lon lat       p1      p2       p3       p4
    1  1 -110 26 0 -110  26 21.68557 21.6072 21.40651 21.27906

Para representar la serie de tiempo podemos re-arreglar nuestros datos
extraídos mediante la siguiente función

``` r
# función para darle formato a datos extraídos con extractPts()
formatXTP <- function(pts, plot.error = FALSE){
  error <- pts$d
  if (plot.error == TRUE){
    x11(); hist(error)
  }
  npts <- nrow(pts)
  ncols <- ncol(pts)
  att <- attributes(pts)
  ans <- data.frame(fecha = att$attribs$period$tmStart, t(pts[, 7:ncols]))
  rownames(ans) <- NULL
  names(ans)[2:(npts+1)] <- paste("loc", 1:npts, sep = ".")
  ans <- ans[order(ans$fecha), ]
  ans
}

tsm <- formatXTP(sst)

head(tsm)
```

             fecha    loc.1
    1   2019-01-01 21.68557
    732 2019-01-01 21.68557
    2   2019-01-02 21.60720
    733 2019-01-02 21.60720
    3   2019-01-03 21.40651
    734 2019-01-03 21.40651

Para graficar la serie de tiempo de temperatura potencial del mar a 0.49
m de profundidad en el punto anterior usamos la función `plot()`.

``` r
plot(tsm, type = "o", pch = 16, col = rgb(1, 0, 0, 0.2))
```

![](Datos_Temporales_en_R_files/figure-commonmark/unnamed-chunk-25-1.png)

Esta serie se puede suavizar con promedios móviles centrados mediante la
función `cma()` del paquete **smooth** que tiene la ventaja de no
producir valores faltantes.

``` r
library(smooth)
```

    Loading required package: greybox

    Package "greybox", v2.0.4 loaded.

    This is package "smooth", v4.2.0
    If you want to know more about the smooth package and forecasting, you can visit my website: https://www.openforecast.org/

``` r
sm <- cma(tsm$loc.1, order = 90)
tsm$sm <- sm$fitted

plot(tsm$fecha, tsm$loc.1, type = "b", pch = 16, col = rgb(1, 0, 0, 0.2),
     xlab = "día", ylab = "temperatura potencial del mar (°C)")
lines(tsm$fecha, tsm$sm, lwd = 2)
```

![](Datos_Temporales_en_R_files/figure-commonmark/unnamed-chunk-26-1.png)

Si quisiéramos obtener los promedios mensuales, podríamos extraer el mes
y el año de la fecha y

``` r
library(lubridate)
```


    Attaching package: 'lubridate'

    The following object is masked from 'package:greybox':

        hm

    The following objects are masked from 'package:base':

        date, intersect, setdiff, union

``` r
tsm$mes <- month(tsm$fecha)
tsm$año <- year(tsm$fecha)
head(tsm)
```

             fecha    loc.1       sm mes  año
    1   2019-01-01 21.68557 21.44759   1 2019
    732 2019-01-01 21.68557 21.44300   1 2019
    2   2019-01-02 21.60720 21.43800   1 2019
    733 2019-01-02 21.60720 21.43200   1 2019
    3   2019-01-03 21.40651 21.42621   1 2019
    734 2019-01-03 21.40651 21.41993   1 2019

Agregar por año mes

``` r
tsm.mens <- aggregate(tsm$loc.1, by = list(mes = tsm$mes, año = tsm$año), mean)
tsm.mens
```

       mes  año        x
    1    1 2019 21.36145
    2    2 2019 19.58896
    3    3 2019 20.51031
    4    4 2019 21.86532
    5    5 2019 24.06840
    6    6 2019 26.31480
    7    7 2019 29.01691
    8    8 2019 30.33844
    9    9 2019 30.38984
    10  10 2019 28.95778
    11  11 2019 26.10605
    12  12 2019 23.13585
    13   1 2020 21.42046
    14   2 2020 19.55499
    15   3 2020 19.63006
    16   4 2020 21.60820
    17   5 2020 25.15369
    18   6 2020 27.14084
    19   7 2020 29.26959
    20   8 2020 29.93487
    21   9 2020 30.79404
    22  10 2020 30.45464
    23  11 2020 25.41389
    24  12 2020 22.42815

``` r
mes <- as.Date(paste(tsm.mens$año, tsm.mens$mes, 15, sep = "-"))
plot(mes, tsm.mens$x,
     xlab = "mes", ylab = "temperatura (°C)", type = "b")
```

![](Datos_Temporales_en_R_files/figure-commonmark/unnamed-chunk-29-1.png)

Si quisieramos personallizar el espaciamiento de las marcas de
graduación en x, por ejemplo cada mes

``` r
rng <- range(mes)
xmes <- seq.Date(rng[1], rng[2], by = "month")
plot(as.Date(paste(tsm.mens$año, tsm.mens$mes, 15, sep = "-")), tsm.mens$x,
     xlab = "", ylab = "temperatura (°C)", type = "b", xaxt = "n")
axis.Date(side = 1, at = mes, labels = format(xmes, format = "%b"))
axis.Date(side = 1, at = xmes[c(1, 13)], labels = c(2019, 2020), line = 2, lwd = 0)
```

![](Datos_Temporales_en_R_files/figure-commonmark/unnamed-chunk-30-1.png)

  
