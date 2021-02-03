# AssignmentV

#load packages
rm(list = ls())

library(jsonlite)
library(httr)
library(rlist)
library(tidyverse)
library(naniar)

# set working directory:
#setwd("")

# set API key
#APIkey <- "7elxdku9GGG5k8j0Xm8KWdANDgecHMV0"
source("Cao_Xiping_AssignmentV.R")

Venues1 <- GET("https://app.ticketmaster.com/discovery/v2/venues?",
                    query =list(apikey = APIkey,
                                local = "*",
                                #page = 2,
                                countryCode = "DE"))
Venues1_content <- fromJSON(rawToChar(Venues1$content))
Venues1_table <- as_tibble(Venues1_content[["_embedded"]][["venues"]])

Venues1_table <- tibble(name = Venues1_table$name,
                        city = Venues1_table$city$name,
                        postalCode = Venues1_table$postalCode,
                        address = Venues1_table$address$line1,
                        url = Venues1_table$url,
                        longitude = as.numeric(Venues1_table$location$longitude),
                        latitude = as.numeric(Venues1_table$location$latitude))

#glimpse(Venues1_table)

pages <- as.numeric(Venues1_content[["page"]][["totalPages"]])
Venues2_table <- list()
for (i in 1:pages) {
Venues2 <- GET("https://app.ticketmaster.com/discovery/v2/venues?",
                    query =list(apikey = APIkey,
                                local = "*",
                                page = i,
                                countryCode = "DE"))
Venues2_content <- fromJSON(rawToChar(Venues2$content))
Venues2_table[[i]] <- as_tibble(Venues2_content[["_embedded"]][["venues"]])
}
Venues3_table <- do.call(bind_rows, Venues2_table)
Venues3_table <- tibble(name = Venues3_table$name,
                        city = Venues3_table$city$name,
                        postalCode = Venues3_table$postalCode,
                        address = Venues3_table$address$line1,
                        url = Venues3_table$url,
                        longitude = as.numeric(Venues3_table$location$longitude),
                        latitude = as.numeric(Venues3_table$location$latitude))
Venues4_table <- bind_rows(Venues3_table, Venues1_table)

#glimpse(Venues4_table)

Venues4_table$longitude[Venues4_table$longitude<5.866944 | Venues4_table$longitude>15.043611] <- NA
Venues4_table$latitude[Venues4_table$latitude < 47.271679 | Venues4_table$latitude> 55.0846] <- NA

ggplot() + 
  geom_polygon(aes(x = long, y = lat, group = group), data = map_data("world", region = "Germany"), fill = "grey90",color = "black") +
  geom_point(aes(x = longitude, y = latitude), data = Venues4_table) +
  theme_void() + 
  coord_quickmap() +
  labs(title = "Event locations across Germany", caption = "Source: ticketmaster.com") + theme(title = element_text(size=8, face='bold'), plot.caption = element_text(face = "italic"))