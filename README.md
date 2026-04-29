# Mapas de Calor de Malaria en Perú

Análisis y visualización geoespacial de casos de malaria en Perú (2009-2024) utilizando datos de vigilancia epidemiológica del sistema RENACE.

## 📊 Descripción del Proyecto

Este proyecto realiza un análisis espacial de los casos de malaria reportados en Perú durante el periodo 2009-2024, generando mapas de calor (coropléticos) que permiten visualizar la distribución geográfica de la enfermedad por departamento.

## 🔧 Requisitos e Instalación

### Paquetes necesarios en R

```r
# Instalar paquetes requeridos
install.packages(c(
  "sf",           # Manejo de datos espaciales
  "tidyverse",    # Manipulación y visualización de datos
  "ggplot2",      # Gráficos y mapas
  "viridis",      # Paletas de colores accesibles
  "patchwork",    # Combinar gráficos
  "readxl"        # Lectura de datos
))

# Librerias en R
library(sf)
library(purrr)
library(tidyverse)
library(ggplot2)
library(ggrepel)
```

## Fuentes de Datos

## Datos de Malaria
- **Archivo**: `datos_abiertos_vigilancia_malaria_2009_2024.csv`
- **Fuente**: Red Nacional de Epidemiología (RENACE)
- **Periodo**: 2009-2024
- **Variables principales**:
  - `departamento`, `provincia`, `distrito`
  - `enfermedad` (MALARIA P. FALCIPARUM / MALARIA POR P. VIVAX)
  - `diagnostico` (B50 / B51)

## Shapefile Departamentos
- **Fuente**: INEI - Geogpsperu
- **Enlace**: https://www.geogpsperu.com



## Carga del shapefile de departamentos
El primer paso es cargar el mapa base de Perú. Para ello, necesitamos un shapefile que contenga las geometrías de los departamentos. Comenzamos definiendo la ruta donde se encuentra este archivo y configurando ese directorio como el lugar de trabajo de R. Luego, verificamos que el archivo con extensión .shp esté efectivamente en la carpeta y lo cargamos usando la función st_read() del paquete sf.

```r
directorio <- "C:/Users/Asus/Desktop/biostats/DEPARTAMENTOS_inei_geogpsperu_suyopomalia"
setwd(directorio)

list.files(directorio, pattern = "\\.shp$", ignore.case = TRUE)
peru_d <- st_read("DEPARTAMENTOS_inei_geogpsperu_suyopomalia.shp")
```

Una vez cargado, podemos inspeccionar su estructura con peru_d y hacer una primera visualización rápida del mapa mudo con ggplot2. Esto nos permite confirmar que el shapefile se ha leído correctamente.

```r
peru_d

ggplot(data = peru_d) +
  geom_sf()
```

## Carga y exploración de los datos de malaria
A continuación, cargamos la base de datos con los casos de malaria reportados entre 2009 y 2024, usando read_csv(). Es importante explorar su contenido para entender qué variables tenemos disponibles y qué valores contienen. Por ejemplo, revisamos las primeras filas, los nombres de las columnas y la distribución de los tipos de enfermedad.

```r
PL <- read_csv("datos_abiertos_vigilancia_malaria_2009_2024.csv")

head(PL)
names(PL)
table(PL$enfermedad)
```

## Procesamiento: conteo de casos por departamento
Agrupamos los registros por departamento y contamos el número total de casos en cada uno. Para ello usamos group_by() y summarise(), y luego ordenamos los resultados de mayor a menor con arrange(). Esto nos permite saber rápidamente qué regiones concentran más casos.

```r
casos_por_depto <- PL %>%
  group_by(departamento) %>%
  summarise(total_casos = n()) %>%
  arrange(desc(total_casos))

print(casos_por_depto)
table(PL$departamento)
```

## Unión del mapa con los datos de casos
Para poder colorear cada departamento según su número de casos, debemos combinar el shapefile (peru_d) con el resumen numérico (casos_por_depto). Usamos left_join() para mantener todos los departamentos del mapa, incluso aquellos que no tengan datos reportados (quedarán como NA). Es fundamental que las columnas con los nombres coincidan; en este caso, la columna del shapefile se llama "NOMBDEP" y la de los datos "departamento".

```r
datos <- peru_d %>% 
  left_join(casos_por_depto, by = c("NOMBDEP" = "departamento"))

head(datos)
```

## Creación del mapa de calor

Finalmente, generamos el mapa coroplético. Usamos geom_sf() y mapeamos el color de relleno (fill) al número total de casos. Elegimos una escala de rojos que va de tonos muy claros (pocos casos) a rojo oscuro (muchos casos), aplicamos una transformación logarítmica para mejorar la visualización de rangos amplios, y establecemos un color gris claro para los departamentos sin datos. Ajustamos también el título, subtítulo, fuente y la estética general del gráfico con theme_minimal() y personalizaciones adicionales.

```r

ggplot(datos) +
  geom_sf(aes(fill = total_casos), color = "white", size = 0.2) +
  labs(
    title = "Distribución de Casos de Malaria en Perú",
    subtitle = "Número total de casos registrados por departamento (2009-2024)",
    caption = "Fuente: Datos abiertos vigilancia malaria (2009-2024)\nElaboración propia",
    fill = "N° de Casos",
    x = NULL,
    y = NULL
  ) +
  scale_fill_gradient(
    name = "N° de Casos",
    low = "#FEE5D9",
    high = "#7F0000",
    trans = "log",
    labels = scales::comma,
    na.value = "gray90"
  ) +
  theme_minimal() +
  theme(
    plot.title = element_text(size = 16, face = "bold", hjust = 0.5),
    plot.subtitle = element_text(size = 11, hjust = 0.5),
    plot.caption = element_text(size = 9, hjust = 0),
    legend.position = "right",
    legend.title = element_text(size = 10, face = "bold")
  )

```


