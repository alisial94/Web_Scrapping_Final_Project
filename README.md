# Web Scrapping Final Project

## Aircraft Data Scrapped for the Project


rm(list=ls())

getwd()
library(rvest)
library(xml2)
library(data.table)
library(esquisse)
library(dplyr)
library(ggplot2)
library(stringr)
library(tidyr)
library(hrbrthemes)
library(ggthemes)
getwd()


### first part getting the link for the main page
p <- read_html("https://aerocorner.com/manufacturers/")
name <- p %>% html_nodes('.list-manufacturers .py-1 .py-1') %>% html_text()
link <- p %>% html_nodes('.list-manufacturers .py-1 .py-1') %>% html_attr('href') 
df <- data.frame('Name' = name, 'link' = link)

### getting the 
get_company <- function(url) {
  l <- read_html(url)
  list_x <- list()
  #entity <- l %>% html_nodes('.entry-title') %>% html_text()
  list_x[['link2']] <- l %>% html_nodes('.py-1 a') %>% html_attr('href')
  list_x[['name2']] <- l %>%  html_nodes('.py-1 a') %>% html_text()
  #list_x[['company']] <- replicate(length(name2), entity)
  return(list_x)
}

url <- df$link
dfx <- lapply(url, get_company)
dfx1 <- rbindlist(dfx, fill = T)

### First Table
get_info <- function(url2) {
  x <- read_html(url2)
  list_y <- list()
  list_y[['name']] <- x %>%  html_nodes('.entry-title') %>% html_text()
  list_y[['manufacturer']] <- x %>%  html_nodes('#main .mb-0:nth-child(2)') %>% html_text()
  list_y[['country']] <- x %>%  html_nodes('#main .mb-0:nth-child(4)') %>% html_text()
  list_y[['manufactured']] <- x %>%  html_nodes('.mb-0:nth-child(6)') %>% html_text()
  list_y[['ICAO']] <- x %>%  html_nodes('.mb-0:nth-child(8)') %>% html_text()
  list_y[['price']] <- x %>%  html_nodes('.mb-0:nth-child(10)') %>% html_text()
  
return(list_y)
}


url2 <- dfx1$link2
list_aircraft <- lapply(url2, get_info)
df_aircraft <- rbindlist(list_aircraft, fill = T)

df_aircraft<- df_aircraft[-c(7, 20, 64, 75, 81, 83, 98, 122, 124, 208, 237, 239,
                             241, 243, 274, 276, 288, 345, 357, 398, 400, 402,
                             429, 470, 500, 502, 504, 506, 528, 531, 533, 535, 537,
                             580, 582, 589, 591, 595, 602, 619, 701, 703, 705, 707,
                             710, 723, 725, 727, 729, 732, 740, 783, 788, 790, 808, 
                             846, 865, 878, 880, 883, 887, 891, 893, 895, 919, 933), ]


### Second Table
get_details <- function(url2) {
  x <- read_html(url2)
  list_z <- list()
  list_z[['name']] <- x %>%  html_nodes('.entry-title') %>% html_text()
  keys2 <- x %>% html_nodes('.col-sm-5') %>% html_text()
  values2 <- x %>% html_nodes('.col-sm-7') %>% html_text()
  for( i in 1:length(keys2)) {
    list_z[[ keys2[i] ]] <- values2[i]
  }
return(list_z)
}

url2 <- dfx1$link2
get_details_list <- lapply(url2, get_details)
df_spec1 <- rbindlist(get_details_list, fill = T)



### Third Table

get_details_1 <- function(url2) {
  x <- read_html(url2)
  list_a <- list()
  list_a[['name']] <- x %>%  html_nodes('.entry-title') %>% html_text()
  keys3 <- x %>% html_nodes('#weights .col-lg-5') %>% html_text()
  values3 <- x %>% html_nodes('#weights .col-lg-7') %>% html_text()
  for( i in 1:length(keys3)) {
    list_a[[ keys3[i] ]] <- values3[i]
  }
  return(list_a)
}

get_details_list_1 <- lapply(url2, get_details_1)
df_spec2 <- rbindlist(get_details_list_1, fill = T)





### Fourth Table

get_details_2 <- function(url2) {
  x <- read_html(url2)
  list_b <- list()
  list_b[['name']] <- x %>%  html_nodes('.entry-title') %>% html_text()
  keys4 <- x %>% html_nodes('#dimensions .col-lg-5') %>% html_text()
  values4 <- x %>% html_nodes('#dimensions .col-lg-7') %>% html_text()
  for( i in 1:length(keys4)) {
    list_b[[ keys4[i] ]] <- values4[i]
  }
  return(list_b)
}

get_details_list_2 <- lapply(url2, get_details_2)
df_spec3 <- rbindlist(get_details_list_2, fill = T)



final_aircraft_data <- cbind(dfx1,df_aircraft,df_spec1,df_spec2,df_spec3)

final_aircraft_data<- final_aircraft_data[, -c(2,9,21,27)]

final_aircraft_data$links <- final_aircraft_data$link2

final_aircraft_data<- final_aircraft_data[, -c(1)]

View(final_aircraft_data)

getwd()
### saving the entire data scrapped into a CSV so it is easier to run the Rmd file
write_csv( final_aircraft_data , 'aircraft_data' )



### calling data from git 

aircraft_data <- read_csv(url("https://raw.githubusercontent.com/alisial94/Web_Scrapping_Final_Project/main/aircraft_data")) 

### Cleaning and Mungging data

### Filtering data
viz_data <- aircraft_data %>%
  filter(!(`Seats - First Class:` %in% "\n ")) %>%
  filter(!(`Wing Tips:` %in% 
             "\n ")) %>%
  filter(!is.na(name))
   
#removing the unwanted columns
viz_data <- viz_data[, c(1,2,3,6,9,13,21)] 


### extracting the price in millions from the string variable
viz_data$Price <- as.numeric(gsub(".*?([0.0000001-9.99]+).*", "\\1", viz_data$price)) 

#converting the power in pound force to horsepower in new column
viz_data$pound_force <- ifelse(grepl('pound-force', viz_data$Power),viz_data$Power,"")
viz_data$pound_force <- gsub('pound-force', '', viz_data$pound_force)
viz_data$pound_force <- as.numeric(gsub(",","",viz_data$pound_force))
viz_data$pound_force <- (viz_data$pound_force/500)*60

#fixing the power that is in horsepower in a new column
viz_data$horse_power <- ifelse(grepl('horsepower', viz_data$Power),viz_data$Power,"")
viz_data$horse_power <- gsub('horsepower', '', viz_data$horse_power)
viz_data$horse_power <- as.numeric(gsub(",","",viz_data$horse_power))


#combining the two new created tables
viz_data$Horsepower <- paste0(viz_data$horse_power, viz_data$pound_force)

#removing the NA in the valuse and saving it as numaric
viz_data$Horsepower <- gsub('NA', '', viz_data$Horsepower)
viz_data$Horsepower <- as.numeric(viz_data$Horsepower)

#fixing the column Fuel Economy
viz_data <- separate(data = viz_data,
                     col = "Fuel Economy:",
                     into = c("nm/g", "km/l"),
                       sep = "/")


viz_data$`nm/g` <- gsub('nautical mile', '', viz_data$`nm/g`)
viz_data$Fuel_Eco <- as.numeric(gsub(",","",viz_data$`nm/g`))


#fixing the column Fuel Tank Capacity
viz_data$fuel_tank_Cap<-gsub("g.*","",viz_data$`Fuel Tank Capacity:`)
viz_data$fuel_tank_Cap <- as.numeric(gsub(",","",viz_data$fuel_tank_Cap))


### Total number of different aircraft each manufacturer produces
Viz1 <-viz_data %>%
  ggplot() +
  aes(x = manufacturer) +
  geom_bar(fill = "#3FD965") +
  labs(x = "Manufacturer", 
       y = "Count of Aircraft ", title = "Aircrafts per Manufacturer") +
  ggthemes::theme_economist_white()+
  theme( axis.text.x = element_text(angle = 60, vjust = 1,
                                    size = 5, hjust = 1))

Viz1



### Linear Regression Price Vs Power
Viz2 <-  viz_data %>% 
  filter(Horsepower <= 10000) %>%
  filter(Price <= 500) %>% 
  ggplot( aes(x = log(Horsepower), y = log(Price))) +
  geom_point(color='red',size=2,alpha=0.6) +
  geom_smooth(method="lm" , formula = y ~ x )+
  labs(x = "Log Horse Power", y = "Log Price (US$ in Million)") +
  ggthemes::theme_economist_white()


Viz2

regPvP <- feols( log(Price) ~ log(Horsepower), data=viz_data, vcov = 'hetero' )
summary( regPvP )


### Linear Regression Fuel Economy Vs Fuel Tank Size
Viz3 <- viz_data %>% 
  filter(fuel_tank_Cap <= 60000) %>%
  filter(Fuel_Eco <= 30) %>% 
  ggplot( aes(x = log(fuel_tank_Cap), y = log(Fuel_Eco))) +
  geom_point(color='red',size=2,alpha=0.6) +
  geom_smooth(method="lm" , formula = y ~ x )+
  labs(x = "Log Fuel Tank Capacity (Gallon)", y = "Log Fuel Economy (nautical mile per gallon)") +
  theme_bw()

Viz3

regEvC <- feols( log(fuel_tank_Cap) ~ log(Fuel_Eco), data=viz_data, vcov = 'hetero' )
summary( regEvC )

