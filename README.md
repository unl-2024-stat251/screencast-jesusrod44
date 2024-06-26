---
title: "Screencast Ramen"
author: "Jesus Rodriguez"
format: html
editor: visual
---

## [Link To Youtube Screencast](https://youtu.be/pn6putf4HHc)

## Time Index

Datasets containing the Top 10 Best and Worst Average Rated Brands: 1:40

Data Cleaning: 4:00

Joining Tables: 16:00

Graphics: 11:00, fix at 20:00

How popular some flavors are, ie: cheese, miso, instant ramen, tonkotsu: 25:00

String Manipulation and some Data Cleaning : 26:30

Reshaping Data: 35:00

Graphics: 50:00

Average Rating of Ramen in Different Countries Around the World: 57:00

Graphics Problems: 1:03:00

Data Manipulation: 1:08:00

Graphics cont. 1:11:30

Finding Problem with USA/United States: 11:22:30

Final World Map Fix: 1:40:00

## Data

```{r}
library(ggplot2)
library(tidyverse)
library(dplyr)



ramen_ratings <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-06-04/ramen_ratings.csv")


#cleaning United States and USA 
ramen_ratings$country <- str_replace(ramen_ratings$country, pattern = "United States", replacement = "USA")

```

## Graphs of most and least rated brands

```{r}

#How many unique brands we have 
length(unique(ramen_ratings$brand))


```

```{r}

#Datasets containing the Top 10 Best and Worst Average Rated Brands

top_brands <- ramen_ratings %>%
  group_by(brand) %>%
  summarize(avg_stars = mean(stars, na.rm = TRUE), total_reviews = n()) %>%
  filter(total_reviews > 10) %>% 
  arrange(desc(avg_stars)) %>% 
  filter(row_number() <= 10)

low_brands <- ramen_ratings %>%
  group_by(brand) %>%
  summarize(avg_stars = mean(stars, na.rm = TRUE), total_reviews = n()) %>%
  filter(total_reviews > 10) %>% 
  arrange(avg_stars) %>% 
  filter(row_number() <= 10)
  
```

```{r}
top_brands<- left_join(top_brands, ramen_ratings)

low_brands <- left_join(low_brands, ramen_ratings)


```

```{r}
library(ggplot2)

#Top 10 Best Graph

ggplot(top_brands, aes(x = stars)) + geom_histogram(binwidth = 0.1) + facet_wrap( ~brand, scales = "free_y")+labs(x = "Ratings", y = "Number of Reviews with That Rating")

#Top 10 Worst Graph

ggplot(low_brands, aes(x = stars)) + geom_histogram(binwidth = 0.1) + facet_wrap( ~brand, scales = "free_y")+labs(x = "Ratings", y = "Number of Reviews with That Rating")


```

## How popular some flavors are, ie: cheese, miso, instant ramen, tonkotsu

```{r}
library(stringr)

#choosing what flavors we want to measure
cheese <- ramen_ratings %>% 
  filter(str_detect(variety, "Cheese")) %>%
  mutate(avg_stars = mean(stars)) %>%
  mutate(flavor = "Cheese")
miso <- ramen_ratings %>% 
  filter(str_detect(variety, "Miso")) %>%
  mutate(avg_stars = mean(stars, na.rm = TRUE))%>%
  mutate(flavor = "Miso")
instant <- ramen_ratings %>% 
  filter(str_detect(variety, "Instant")) %>%
  mutate(avg_stars = mean(stars, na.rm = TRUE))%>%
  mutate(flavor = "Instant")
tonkotsu <- ramen_ratings %>% 
  filter(str_detect(variety, "Tonkotsu")) %>%
  mutate(avg_stars = mean(stars, na.rm = TRUE))%>%
  mutate(flavor = "Tonkotsu")
chicken <- ramen_ratings %>% 
  filter(str_detect(variety, "Chicken")) %>%
  mutate(avg_stars = mean(stars, na.rm = TRUE))%>%
  mutate(flavor = "Chicken")
shrimp <- ramen_ratings %>% 
  filter(str_detect(variety, "Shrimp")) %>%
  mutate(avg_stars = mean(stars, na.rm = TRUE))%>%
  mutate(flavor = "Shrimp")
spicy <- ramen_ratings %>% 
  filter(str_detect(variety, "Spicy")) %>%
  mutate(avg_stars = mean(stars, na.rm = TRUE))%>%
  mutate(flavor = "Spicy")
mushroom <- ramen_ratings %>% 
  filter(str_detect(variety, "Mushroom")) %>%
  mutate(avg_stars = mean(stars, na.rm = TRUE))%>%
  mutate(flavor = "Mushroom")


#binding rows of all flavor datasets together into one
flavors <- bind_rows(cheese, miso, instant, tonkotsu, chicken, shrimp, spicy, mushroom)

```

```{r}

#Sort flavor by avg_stars
#use reorder

flavors <- flavors %>% 
  mutate(flavor = reorder(flavor, avg_stars))


ggplot(flavors, aes(x = avg_stars, y = flavor, fill = flavor )) + geom_col() + xlim(0,5) + labs(x = "Rating", y = "Flavor", title = "How popular some flavors are, ie: cheese, miso, instant ramen, tonkotsu")+guides(fill = "none")

```

## Plot showing average rating of ramen from all countries

```{r}
library(mapdata)
library(ggthemes)

world_coordinates <- map_data("world")
countries <- map_data("world")

```

```{r}

#avg rating of all countries
top_countries <- ramen_ratings %>%
  group_by(country) %>%
  summarize(avg_stars = mean(stars, na.rm = TRUE), total_reviews = n()) %>%
  arrange(desc(avg_stars))

#43 unique countries
length(unique(ramen_ratings$country))

countries <- countries %>% rename(country = region)
c <- left_join(top_countries, countries)




ggplot(countries, aes(x = long, y = lat, group = group)) + 
  geom_polygon(color = "black", fill = "white") + theme_map() + 
  geom_polygon(data = c, aes(fill = avg_stars)) + scale_fill_gradient_tableau("Blue-Green Sequential") + labs(title = "Average Rating of Ramen in Different Countries Around the World")


c10 <- c %>%
  filter(total_reviews >=10)

ggplot(countries, aes(x = long, y = lat, group = group)) + 
  geom_polygon(color = "black", fill = "white") + theme_map() + 
  geom_polygon(data = c10, aes(fill = avg_stars)) + scale_fill_gradient_tableau("Blue-Green Sequential") + labs(title = "Average Rating of Ramen in Different Countries Around the World with More than 10 Reviews")


```
