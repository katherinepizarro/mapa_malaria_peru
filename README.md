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



## Carga de archivo Shape

# Definir la ruta donde están tus archivos
```r
directorio <- "C:/Users/Asus/Desktop/biostats/DEPARTAMENTOS_inei_geogpsperu_suyopomalia"
```

# Cambiar al directorio de trabajo
```r
setwd(directorio)
```

# Verificar qué archivos .shp existen en la carpeta
```r
list.files(directorio, pattern = "\\.shp$", ignore.case = TRUE)
```

# Cargar el shapefile
```r
peru_d <- st_read("DEPARTAMENTOS_inei_geogpsperu_suyopomalia.shp")
```

# Ver la estructura del shapefile
```r
peru_d
```

# Graficar el mapa básico
```
ggplot(data = peru_d) +
  geom_sf()
```

# Cargar datos de casos de malaria
```r
PL <- read_csv("datos_abiertos_vigilancia_malaria_2009_2024.csv")
```

# Ver las primeras filas
```r
head(PL)
```

# Ver los nombres de las columnas
```r
names(PL)
```

# Ver valores únicos en enfermedad
```r
table(PL$enfermedad)
```

# Calcular total de casos por departamento
```r
casos_por_depto <- PL %>%
  group_by(departamento) %>%           # Agrupar por departamento
  summarise(
    total_casos = n()                  # Contar número de filas (casos)
  ) %>%
  arrange(desc(total_casos))           # Ordenar de mayor a menor
```

# Ver los resultados
```r
print(casos_por_depto)
```

# Ver tabla de frecuencias
```r
table(PL$departamento)
```

# Unir shapefile con datos de casos
```r
datos <- peru_d %>% 
  left_join(casos_por_depto, by = c("NOMBDEP" = "departamento"))
```

# Verificar la unión
```r
head(datos)
```

# Versión 1: Gradiente de rojos (recomendada para malaria)
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
    low = "#FEE5D9",    # Rojo claro (pocos casos)
    high = "#7F0000",   # Rojo oscuro (muchos casos)
    trans = "log",      # Escala logarítmica
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
