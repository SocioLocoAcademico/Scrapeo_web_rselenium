# Hola, se adjunta el código utilizado en una de las prácticas realizadas en la asignatura Investigación en Red, en el contexto del Master de Metodología de Investigación Social.

# Espero que les sirva de base para futuras aplicaciones.

Instalación de las librerías, una vez instaladas este proceso se puede omitirPara omitir, o borrar o colocar # delante

installed.packages(RSelenium)
installed.packages(tidyverse)
installed.packages(lubridate)
installed.packages(rvest)
installed.packages(openxlsx)
install.packages("remotes")
install.packages("webdriver")

# Librerias a utilizar

library(RSelenium)
library(tidyverse)
library(lubridate)
library(rvest)
library(openxlsx)
library(remotes)
library(webdriver)

### Llaves para mapear la pantalla

selKeys <- list(
  null = "\uE000",
  cancel = "\uE001",
  help = "\uE002",
  backspace = "\uE003",
  tab = "\uE004",
  clear = "\uE005",
  return = "\uE006",
  enter = "\uE007",
  shift = "\uE008",
  control = "\uE009",
  alt = "\uE00A",
  pause = "\uE00B",
  escape = "\uE00C",
  space = "\uE00D",
  page_up = "\uE00E",
  page_down = "\uE00F",
  end = "\uE010",
  home = "\uE011",
  left_arrow = "\uE012",
  up_arrow = "\uE013",
  right_arrow = "\uE014",
  down_arrow = "\uE015",
  insert = "\uE016",
  delete = "\uE017",
  semicolon = "\uE018",
  equals = "\uE019",
  numpad_0 = "\uE01A",
  numpad_1 = "\uE01B",
  numpad_2 = "\uE01C",
  numpad_3 = "\uE01D",
  numpad_4 = "\uE01E",
  numpad_5 = "\uE01F",
  numpad_6 = "\uE020",
  numpad_7 = "\uE021",
  numpad_8 = "\uE022",
  numpad_9 = "\uE023",
  multiply = "\uE024",
  add = "\uE025",
  separator = "\uE026",
  subtract = "\uE027",
  decimal = "\uE028",
  divide = "\uE029",
  f1 = "\uE031",
  f2 = "\uE032",
  f3 = "\uE033",
  f4 = "\uE034",
  f5 = "\uE035",
  f6 = "\uE036",
  f7 = "\uE037",
  f8 = "\uE038",
  f9 = "\uE039",
  f10 = "\uE03A",
  f11 = "\uE03B",
  f12 = "\uE03C",
  command_meta = "\uE03D"
)

# Iniciar el servidor de Selenium
rD <- rsDriver(browser = "firefox", chromever = NULL)
remDr <- rD$client

# Entrar en la página que se desea
remDr$navigate("PAGINA_WEB")

# Encontrar un elemento concreto y clicar
webElem <- remDr$findElement(using = "css selector", value = "button.VfPpkd-LgbsSe")
webElem$clickElement()

# Esperar a que la página se cargue completamente (puedes ajustar el tiempo según sea necesario)
remDr$wait(10)

# Leer toda la pagina

html_obj <- remDr$getPageSource(header = TRUE)[[1]] %>% read_html()

# Numero de reviews para contar los scrollers 

reviews_full <- html_obj %>% html_elements("div.F7nice") %>% html_text()

# Utiliza la strsplit para dividir caracteres bajo un formato concreto

reviews_full_format<- strsplit(reviews_full, "[()]")[[1]]

# Formateo del número de reviews para establecer el repeat

reviews_number <- as.numeric(reviews_full_format[2])

# Mover el raton hasta reviews y clickar, como sumar y restar divs

webElem <- remDr$findElement(using = "css selector", value = "button.G7m0Af+ .hh2c6")
webElem$clickElement()

Sys.sleep(1)

# Capturar el tiempo al inicio del proceso

tiempo_inicial_scroll <- Sys.time()

# Mover el raton hasta otra zona donde dejar el raton y bajar las reseñas con "end" y finalizar repeat

webElem <- remDr$findElement(using = "css selector", value = "button.g88MCb")

#Este código lo he utilizado para hacer pruebas estableciendo un límite antes de "saturar la memoria"
#maximum_lenght_reviews <- 1300

repeat {
  webElem$sendKeysToElement(list(key = "end"))
  html_obj <- remDr$getPageSource(header = TRUE)[[1]] %>% read_html()
  usuarios <- html_obj %>% html_elements("div.d4r55") %>% html_text()
  reviews <- html_obj %>% html_elements("div.MyEned") %>% html_text()
  Sys.sleep(1)

  #Este if es aquel que pudimos reflexionar juntos la última vez que te pregunté, se ordena primero las que tiene comentarios, por lo que cuando llega a la que no tiene, rompe
  if (length(usuarios) > length(reviews))
    
    break 
}

# Este código lo sustituyo por el anterior if, para que rompa en aquellas páginas donde el número de reviews es superior a 1300
#if (length(reviews) >= maximum_lenght_reviews | (length(usuarios) > length(reviews))

Sys.sleep(1)

# Capturar el tiempo al final del proceso
tiempo_final_scroll <- Sys.time()

# Calcular la duración del proceso
duracion_scroll <- tiempo_final_scroll - tiempo_inicial_scroll

# Capturar el tiempo al inicio del proceso

tiempo_inicial_abrir <- Sys.time()

# Una vez sroleado todas las reviews, se clican todos los "Abrir", para mostrar review completa

puede_realizar_accion <- TRUE

# Crear un bucle while que se repita mientras se pueda realizar la acción

while (puede_realizar_accion) {
  
  # Intentar realizar la acción dentro del bloque tryCatch
  
  tryCatch({
    
    # Encontrar abrir y clicar
    
    webElem_mas_button <- remDr$findElement(using = "css selector", value = "button.w8nwRe")
    webElem_mas_button$clickElement()
    
  }, error = function(e) {
    
    # Se produce un error cuando no encuentra más, por lo que el bucle salta
    
    puede_realizar_accion <<- FALSE
  })
}

Sys.sleep(1)

# Capturar el tiempo al final del proceso

tiempo_final_abrir <- Sys.time()

# Calcular la duración del proceso

duracion_abrir <- tiempo_final_abrir - tiempo_inicial_abrir

# Leemos toda la página de nuevo para recoger la info a scrapear

html_obj <- remDr$getPageSource(header = TRUE)[[1]] %>% read_html()

# Recogemos usuarios, fechad de publiación y reviews

usuarios <- html_obj %>% html_elements("div.d4r55") %>% html_text() 
data <- html_obj %>% html_elements("span.rsqaWe") %>% html_text()
reviews <- html_obj %>% html_elements("div.MyEned") %>% html_text()

# Existe el problema de que alguas review son sin texto y no coinciden las filas para hacer una tabla
# Se acortan con el supuesto de que se ordenan primero las que tienen texto

usuarios <- usuarios[1:length(reviews)]
data <- data[1:length(reviews)]

tabla_datos_FINAL <- data.frame(usuarios, data, reviews)

# Crear un vector de fecha que corresponda al número de filas en 'Resultados'

fecha_de_scrapeo <- rep(Sys.Date(), nrow(tabla_datos_FINAL))

# Agregar el vector de fechas al data frame 'Resultados'

tabla_datos_FINAL <- data.frame(usuarios, fecha_de_scrapeo, data, reviews)

Sys.sleep(1)

# Cerrar el servidor

rD[["server"]]$stop()

# Escribir la tabla de datos en un archivo xls

write.xlsx(tabla_datos_FINAL, "NUEVO.xlsx")

# Capturar el tiempo al final del proceso
tiempo_final <- Sys.time()

# Calcular la duración del proceso
duracion_total <- tiempo_final - tiempo_inicial

# Imprimir la duración del último proceso
print(duracion_total)
print(duracion_scroll)
print(duracion_abrir)
