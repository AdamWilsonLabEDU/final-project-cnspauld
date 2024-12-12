---
title: "Comparing Water Health in a Pre and Post COVID Landscape"
author: Caroline Spaulding
subtitle: A Study in in Buffalo, NY
date: today
date-format: long
format: pdf
---

# Introduction

Many factors can affect how fresh water quality and composition is affected. Some of these factors include, but are not limited to, pH and turbidity. The pH of water measures how acidic or basic it may be on a scale of 0 to 14, with 0 representing a solution that is highly acidic, and 14 representing a solution that is highly basic. 7 represents a solution that is neutral. In a healthy freshwater body, pH typically ranges from 6.5 to 8.5 (Hem, 2001). Turbidity is essentially the measure of how clear a liquid is. The cloudier it is, typically the unhealthier the water may be. This usually means the water contains pollutants, or particulate matter such as silt and clay. Turbidity is measured in Nephelometric Turbidity Units (NTUs). High NTU values indicate a high level of pollutants or particulates in the water, while low turbidity values indicate clearer water. Healthy drinking water is usually present with a turbidity value of 1 or less (World Health Organization, 2017). In this project, I will analyze the differences from the years 2019 and 2023 to better understand how the COVID-19 lock down affected essential measures of water quality in the Buffalo area. I hypothesize that water quality will have improved during the aftermath of the pandemic. This is because human activity was greatly reduced for a long period of time, with only essential personnel leaving their homes. With no human activity to contribute to the addition of pollutants into the water system for most of 2020, pH and turbidity should stabilize to healthy levels.

# Materials and methods

The packages used in this project are ggplot2, sf, leaflet, tidyverse, readxl, ggpubr, plotly, dplyr, lubridate, and zoo. All of my data is courtesy of the Buffalo Niagara Waterkeeper. According to their website, their mission is as follows, ". . . we PROTECT the water, we RESTORE both the waterways and the surrounding ecosystems, we CONNECT people to their waterways, and we INSPIRE both economic activity along the waterways and community engagement". Their mission aligns with many of my own values surrounding environmental health and prosperity, which is why I chose to work with their data. They have issued many water quality reports, allowing people to access pertinent information on their local environment.

ggplot2 is an R package that utilizes and underlying grammar, called the Grammar of Graphics. It allows you to create graphs and display data based on the combination of independent components (Ggplot2: Elegant Graphics for Data Analysis (3e) - 1  Introduction, n.d.). sf is an R package that represents simple features and contain at least one geometry list-column and are a variable in a data frame or a tibble (Spatial Data Science - 7  Introduction to Sf and Stars, 2016). leaflet is an R package that uses an open-source JavaScript library to create interactive and dynamic online maps (Hahn, n.d.). The tidyverse is an R package that is designed to easily install multiple packages at once that share the same API design and similar functions, including but not limited to ggplot2 and readxl (Hadley Wickham, 2017). The readxl package in R is a relatively simple one; it is used to read data in Excel files (How to Use Readxl Package to Read Data in R, 2022). ggpubr is an R package that expands upon the functions in ggplot2, allowing for the combination of multiple plots and the additon of annotations and captions (Ggpubr in R, 2023). plotly is an R package that uses the open source JavaScript library to create interactive web-based graphs (Getting Started with Plotly in R). dplyr is an R package that allows for the creation of quick and easy analysis, particularly for creating summary statistics (Phillips, 2024). lubridate is an R package that allows for the transformation of columns into date objects (Working with Dates and Times in R Using the Lubridate Package | UVA Library, n.d.). zoo is an R package that allows for easy use of time-series data (GeeksforGeeks, 2024). 

# Data Locations and Visualization

```{r}
library(ggplot2)
library(sf)
library(leaflet)
library(tidyverse)
library(readxl)

waterkeeper_data_2019 <- read.csv("data/WaterKeeper_2019.csv")
# Convert the data into a spatial object (sf)
waterkeeper_data_2019_sf <- st_as_sf(waterkeeper_data_2019, coords = c("LONG", "LAT"), crs = 4326)
waterkeeper_data_2019_sf$LONG <- waterkeeper_data_2019$LONG 
waterkeeper_data_2019_sf$LAT <- waterkeeper_data_2019$LAT 
# Create an interactive leaflet map
leaflet(waterkeeper_data_2019_sf) %>%
  addTiles() %>%  # Add OpenStreetMap tiles as a base layer
  addCircleMarkers(
    ~LONG, ~LAT,     # Latitude and Longitude columns
    radius = 5,      # Size of the markers
    color = "blue",  # Marker color
    popup = ~Location      # Pop-up text (display the "id" column when you click a marker)
  ) %>%
  setView(lng = mean(waterkeeper_data_2019$LONG), lat = mean(waterkeeper_data_2019$LAT), zoom = 10)
```

The code above represents data gathered by the Buffalo Niagara Waterkeeper in 2019. The plot is labelled by location, so one can see what monitoring stations the water data was collected from.

```{r}
library(ggplot2)
library(sf)
library(leaflet)
library(readxl)

waterkeeper_data_2023 <- read_excel("data/2023_Riverwatch_Data.xlsx")

#Convert the data into a spatial object
waterkeeper_data_2023_sf <- st_as_sf(waterkeeper_data_2023, coords = c("longitude", "latitude"), crs = 4326)

waterkeeper_data_2023_sf$longitude <- waterkeeper_data_2023$longitude 
waterkeeper_data_2023_sf$latitude <- waterkeeper_data_2023$latitude

leaflet(waterkeeper_data_2023_sf) %>%
  addTiles() %>%
  addCircleMarkers(
    ~longitude, ~latitude,
    radius = 5,
    color = "red",
    popup = ~station_name
  ) %>%
  setView(lng = mean(waterkeeper_data_2023$longitude), lat = mean(waterkeeper_data_2023$latitude), zoom = 10)
```

The code above shows data collected by the Buffalo Niagara Waterkeeper in 2023. This plot is also labelled by location, so one can see what body of water the data was collected from.

## Download and clean all required data

```{r}
library(ggplot2)
library(sf)

waterkeeper_data_2019 <- read.csv("data/WaterKeeper_2019.csv")
# Convert the data into a spatial object (sf)
waterkeeper_data_2019_sf <- st_as_sf(waterkeeper_data_2019, coords = c("LONG", "LAT"), crs = 4326)
waterkeeper_data_2019_sf$LONG <- waterkeeper_data_2019$LONG 
waterkeeper_data_2019_sf$LAT <- waterkeeper_data_2019$LAT 
waterkeeper_data_2019_st <- waterkeeper_data_2019_sf %>% 
  mutate(Month = factor(Month, levels = rev(month.name), ordered = TRUE))

library(tidyverse)

t_range <- range(waterkeeper_data_2019_sf$turbidity, na.rm = TRUE)

# Plotting turbidity over time for Waterkeeper 2019 data
plot_turbidity_2019 <- ggplot(waterkeeper_data_2019_st, aes(x = turbidity, y = Month, group = Month, color = Month)) +
  geom_line(linewidth = 1) +  # Line plot
  geom_point(linewidth = 2) +  # Scatter plot points
  scale_x_continuous(limits = t_range) +  # Make sure t_range is correctly defined
  labs(x = "Turbidity (NTU)", y = "Month", title = "Turbidity vs Month in 2019") +  # Add labels and title
  theme_minimal() + 
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1)  # Rotate x-axis labels
  )
```
The code above represents my data import and clean up for turbidity levels in 2019.

```{r}
library(ggplot2)
library(sf)
library(readxl)

waterkeeper_data_2023 <- read_excel("data/2023_Riverwatch_Data.xlsx")
#Convert the data into a spatial object
waterkeeper_data_2023_sf <- st_as_sf(waterkeeper_data_2023, coords = c("longitude", "latitude"), crs = 4326)

waterkeeper_data_2023_sf$longitude <- waterkeeper_data_2023$longitude 
waterkeeper_data_2023_sf$latitude <- waterkeeper_data_2023$latitude

waterkeeper_data_2023_st <- waterkeeper_data_2023_sf %>%
  mutate(
    Month = format(collection_date, "%B"),  # Extract month names
    Month = factor(Month, levels = rev(month.name), ordered = TRUE)  # Reverse the month order
  )

library(tidyverse)
  
t_range2 <- range(waterkeeper_data_2023_sf$`Turbidity [p:2922]`)
  
# Plot turbidity values, excluding NA values for waterkeeper 2023 data
plot_turbidity_2023 <- ggplot(waterkeeper_data_2023_st, aes(x = `Turbidity [p:2922]`, y = Month, group = Month, color = Month)) +
  geom_line(linewidth = 1, na.rm = TRUE) +  # Exclude NA values for lines
  geom_point(linewidth = 2, na.rm = TRUE) +  # Exclude NA values for points
  scale_x_continuous(limits = c(min(waterkeeper_data_2023_st$`Turbidity [p:2922]`, na.rm = TRUE),
                                max(waterkeeper_data_2023_st$`Turbidity [p:2922]`, na.rm = TRUE))) +  # Exclude NA values for x-axis limits
  labs(x = "Turbidity (NTU)", y = "Month", title = "Turbidity vs Month in 2023") + 
  theme_minimal() + 
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

```
The code above represents my data import and clean up for turbidity levels in 2023.

```{r}
library(ggplot2)
library(sf)
library(readxl)
library(tidyverse)

waterkeeper_data_2019 <- read.csv("data/WaterKeeper_2019.csv")
# Convert the data into a spatial object (sf)
waterkeeper_data_2019_sf <- st_as_sf(waterkeeper_data_2019, coords = c("LONG", "LAT"), crs = 4326)
waterkeeper_data_2019_sf$LONG <- waterkeeper_data_2019$LONG 
waterkeeper_data_2019_sf$LAT <- waterkeeper_data_2019$LAT 
waterkeeper_data_2019_st <- waterkeeper_data_2019_sf %>% 
  mutate(Month = factor(Month, levels = rev(month.name), ordered = TRUE))

pH_range <- range(waterkeeper_data_2019_sf$pH)

#Plot pH values over time for 2019 waterkeeper data
plot_pH_2019 <- ggplot(waterkeeper_data_2019_st, aes(x = pH, y = Month, group = Month, color = Month)) +
  geom_line(size = 1) +  # Line plot
  geom_point(size = 2) +  # Scatter plot points
  scale_x_continuous(limits = pH_range) +  
  labs(x = "pH", y = "Month", title = "pH vs Month in 2019") +  # Add labels and title
  theme_minimal() + 
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1)  # Rotate x-axis labels
  )
```

The code above represents my data import and clean up for pH levels in 2019.

```{r}
library(ggplot2)
library(sf)
library(readxl)
library(tidyverse)
waterkeeper_data_2023 <- read_excel("data/2023_Riverwatch_Data.xlsx")
#Convert the data into a spatial object
waterkeeper_data_2023_sf <- st_as_sf(waterkeeper_data_2023, coords = c("longitude", "latitude"), crs = 4326)

waterkeeper_data_2023_sf$longitude <- waterkeeper_data_2023$longitude 
waterkeeper_data_2023_sf$latitude <- waterkeeper_data_2023$latitude

waterkeeper_data_2023_st <- waterkeeper_data_2023_sf %>%
  mutate(
    Month = format(collection_date, "%B"),  # Extract month names
    Month = factor(Month, levels = rev(month.name), ordered = TRUE)  # Reverse the month order
  )

pH_range2 <- range(waterkeeper_data_2023_sf$`pH [p:2923]`)

#Plot pH values over time for 2023 waterkeeper data
plot_pH_2023 <- ggplot(waterkeeper_data_2023_st, aes(x = `pH [p:2923]`, y = Month, group = Month, color = Month)) +
  geom_line(size = 1, na.rm = TRUE) +  # Exclude NA values for lines
  geom_point(size = 2, na.rm = TRUE) +  # Exclude NA values for points
  scale_x_continuous(limits = c(min(waterkeeper_data_2023_st$`pH [p:2923]`, na.rm = TRUE),
                                max(waterkeeper_data_2023_st$`pH [p:2923]`, na.rm = TRUE))) +  # Exclude NA values for x-axis limits
  labs(x = "pH", y = "Month", title = "pH vs Month in 2023") + 
  theme_minimal() + 
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

The code above represents my data import and clean up for pH levels in 2023.

The data import and clean up for all of the 2019 data was relatively simple. All of the data was labelled in a clear and concise way, with every point containing a value. The 2023 data was a bit more complex. The import itself was simple, however the column names contained spaces and special characters that R was having trouble reading. Additionally, some rows contained NA values, which had to be filtered out in order for the data to display properly.

# Results

Below are the results of my analysis, with turbidity and pH values presented side by side for easy comparison, as well as on a combined plotly graph for a more interactive analysis. It took me a while to get to these steps. I had no trouble displaying the geographic coordinates on a leaflet map, allowing viewers to get a better understanding and grasp of the study area. I was able to do this by loading the data into R utilizing the readxl package to do so. I then converted the data into a spatial object in order to display the data both graphically and geographically. In order for the data to display, I had to ensure the latitude and longitude values were assigned correctly by ensuring the projection was WGS84. However, displaying the data on a ggplot graph was a bit more complex. The data was displaying in reverse chronological order, so I used the mutate function to reverse the order so it would appear chronologically. I used the range function for both the turbidity and pH plots in order to ensure the x-axis was scaled correctly. I created a line plot to represent turbidity and pH values in a given month in 2019 and again in 2023. Lastly, for the ggplot portion of the analysis, I wanted to display the turbidity and pH values in each year side by side for a more easily readable comparison. I did this with the ggarrange function from the ggpubr package. This way the values for before and after COVID can be seen graphically.

```{r}
library(ggplot2)
library(sf)
library(readxl)

waterkeeper_data_2019 <- read.csv("data/WaterKeeper_2019.csv")
# Convert the data into a spatial object (sf)
waterkeeper_data_2019_sf <- st_as_sf(waterkeeper_data_2019, coords = c("LONG", "LAT"), crs = 4326)
waterkeeper_data_2019_sf$LONG <- waterkeeper_data_2019$LONG 
waterkeeper_data_2019_sf$LAT <- waterkeeper_data_2019$LAT 
waterkeeper_data_2019_st <- waterkeeper_data_2019_sf %>% 
  mutate(Month = factor(Month, levels = rev(month.name), ordered = TRUE))

library(tidyverse)

t_range <- range(waterkeeper_data_2019_sf$turbidity, na.rm = TRUE)

# Plotting turbidity over time for Waterkeeper 2019 data
plot_turbidity_2019 <- ggplot(waterkeeper_data_2019_st, aes(x = turbidity, y = Month, group = Month, color = Month)) +
  geom_line(linewidth = 1) +  # Line plot
  geom_point(linewidth = 2) +  # Scatter plot points
  scale_x_continuous(limits = t_range) +  # Make sure t_range is correctly defined
  labs(x = "Turbidity (NTU)", y = "Month", title = "Turbidity vs Month in 2019") +  # Add labels and title
  theme_minimal() + 
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1)  # Rotate x-axis labels
  )

library(ggplot2)
library(sf)
library(readxl)

waterkeeper_data_2023 <- read_excel("data/2023_Riverwatch_Data.xlsx")
#Convert the data into a spatial object
waterkeeper_data_2023_sf <- st_as_sf(waterkeeper_data_2023, coords = c("longitude", "latitude"), crs = 4326)

waterkeeper_data_2023_sf$longitude <- waterkeeper_data_2023$longitude 
waterkeeper_data_2023_sf$latitude <- waterkeeper_data_2023$latitude

waterkeeper_data_2023_st <- waterkeeper_data_2023_sf %>%
  mutate(
    Month = format(collection_date, "%B"),  # Extract month names
    Month = factor(Month, levels = rev(month.name), ordered = TRUE)  # Reverse the month order
  )

library(tidyverse)
  
t_range2 <- range(waterkeeper_data_2023_sf$`Turbidity [p:2922]`)
  
# Plot turbidity values, excluding NA values for waterkeeper 2023 data
plot_turbidity_2023 <- ggplot(waterkeeper_data_2023_st, aes(x = `Turbidity [p:2922]`, y = Month, group = Month, color = Month)) +
  geom_line(linewidth = 1, na.rm = TRUE) +  # Exclude NA values for lines
  geom_point(linewidth = 2, na.rm = TRUE) +  # Exclude NA values for points
  scale_x_continuous(limits = c(min(waterkeeper_data_2023_st$`Turbidity [p:2922]`, na.rm = TRUE),
                                max(waterkeeper_data_2023_st$`Turbidity [p:2922]`, na.rm = TRUE))) +  # Exclude NA values for x-axis limits
  labs(x = "Turbidity (NTU)", y = "Month", title = "Turbidity vs Month in 2023") + 
  theme_minimal() + 
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

library(ggpubr)

#Turbidity plots side by side
ggarrange(
  plot_turbidity_2019, plot_turbidity_2023,
  ncol = 2, 
  nrow = 2,
  common.legend = TRUE,   # Add common legend if needed
  legend = "bottom",      # Place legend at the bottom
  align = "hv",           # Align the plots horizontally and vertically
  labels = c("Before COVID", "After COVID" ),  # Label the plots
  label.x = 0.5,                     # Center labels horizontally
  label.y = -0.15,                   # Move labels below the plots
  hjust = 0.5,                       # Center the labels horizontally
  vjust = 1, 
  font.label = list(size = 14)      # Customize the label size
)
```

```{r}
library(ggplot2)
library(sf)
library(readxl)
library(tidyverse)

waterkeeper_data_2019 <- read.csv("data/WaterKeeper_2019.csv")
# Convert the data into a spatial object (sf)
waterkeeper_data_2019_sf <- st_as_sf(waterkeeper_data_2019, coords = c("LONG", "LAT"), crs = 4326)
waterkeeper_data_2019_sf$LONG <- waterkeeper_data_2019$LONG 
waterkeeper_data_2019_sf$LAT <- waterkeeper_data_2019$LAT 
waterkeeper_data_2019_st <- waterkeeper_data_2019_sf %>% 
  mutate(Month = factor(Month, levels = rev(month.name), ordered = TRUE))

pH_range <- range(waterkeeper_data_2019_sf$pH)

#Plot pH values over time for 2019 waterkeeper data
plot_pH_2019 <- ggplot(waterkeeper_data_2019_st, aes(x = pH, y = Month, group = Month, color = Month)) +
  geom_line(linewidth = 1) +  # Line plot
  geom_point(linewidth = 2) +  # Scatter plot points
  scale_x_continuous(limits = pH_range) +  
  labs(x = "pH", y = "Month", title = "pH vs Month in 2019") +  # Add labels and title
  theme_minimal() + 
  theme(
    axis.text.x = element_text(angle = 45, hjust = 1)  # Rotate x-axis labels
  )

library(ggplot2)
library(sf)
library(readxl)
library(tidyverse)
waterkeeper_data_2023 <- read_excel("data/2023_Riverwatch_Data.xlsx")
#Convert the data into a spatial object
waterkeeper_data_2023_sf <- st_as_sf(waterkeeper_data_2023, coords = c("longitude", "latitude"), crs = 4326)

waterkeeper_data_2023_sf$longitude <- waterkeeper_data_2023$longitude 
waterkeeper_data_2023_sf$latitude <- waterkeeper_data_2023$latitude

waterkeeper_data_2023_st <- waterkeeper_data_2023_sf %>%
  mutate(
    Month = format(collection_date, "%B"),  # Extract month names
    Month = factor(Month, levels = rev(month.name), ordered = TRUE)  # Reverse the month order
  )

pH_range2 <- range(waterkeeper_data_2023_sf$`pH [p:2923]`)

#Plot pH values over time for 2023 waterkeeper data
plot_pH_2023 <- ggplot(waterkeeper_data_2023_st, aes(x = `pH [p:2923]`, y = Month, group = Month, color = Month)) +
  geom_line(linewidth = 1, na.rm = TRUE) +  # Exclude NA values for lines
  geom_point(linewidth = 2, na.rm = TRUE) +  # Exclude NA values for points
  scale_x_continuous(limits = c(min(waterkeeper_data_2023_st$`pH [p:2923]`, na.rm = TRUE),
                                max(waterkeeper_data_2023_st$`pH [p:2923]`, na.rm = TRUE))) +  # Exclude NA values for x-axis limits
  labs(x = "pH", y = "Month", title = "pH vs Month in 2023") + 
  theme_minimal() + 
  theme(axis.text.x = element_text(angle = 45, hjust = 1))

library(ggpubr)

#pH plots side by side
ggarrange(
  plot_pH_2019, plot_pH_2023,
  ncol = 2, 
  nrow = 2,
  common.legend = TRUE,   # Add common legend if needed
  legend = "bottom",      # Place legend at the bottom
  align = "hv",           # Align the plots horizontally and vertically
  labels = c("Before COVID", "After COVID" ),  # Label the plots
  label.x = 0.5,                     # Center labels horizontally
  label.y = -0.15,                   # Move labels below the plots
  hjust = 0.5,                       # Center the labels horizontally
  vjust = 1,
  font.label = list(size = 14)      # Customize the label size
)
```

After creating the ggplot graphs, I wanted to create something a bit more interactive, so I turned to the plotly package. This took a very long time, but now it's very easy to look at both the turbidity and pH trends over time for the same year. The biggest issues I ran into were with the date values on both sets of data. For the 2019 data, the date value was in a character format, so I had to convert that to a date format in order for the data to display in a plotly format. The 2023 date was in a datetime format, so I had to filter that out and convert the POSXIT format to a regular date format. I then proceeded to aggregate the data to a weekly basis, as there was too much data to include everything without the graph looking too busy. This is when I used the lubridate package. To preserve the accuracy of the data while reducing the data size, I calculated the weekly average turbidity and pH and ensured all NA values were excluded using the na.rm = TRUE argument. I then applied a 5-period moving average to smooth the data in order to reduce noise in the time-series, avoiding the affect of short-term fluctuations. I made sure to create two different y-axes in order to properly represent turbidity and pH, as they have different scales. These steps ensure that the overall trend of each year is represented accurately without unnecessary overcrowding of data.

```{r}
library(ggplot2)
library(sf)
library(readxl)
library(tidyverse)

waterkeeper_data_2019 <- read.csv("data/WaterKeeper_2019.csv")
# Convert the data into a spatial object (sf)
waterkeeper_data_2019_sf <- st_as_sf(waterkeeper_data_2019, coords = c("LONG", "LAT"), crs = 4326)
waterkeeper_data_2019_sf$LONG <- waterkeeper_data_2019$LONG 
waterkeeper_data_2019_sf$LAT <- waterkeeper_data_2019$LAT 
waterkeeper_data_2019_st <- waterkeeper_data_2019_sf %>% 
  mutate(Month = factor(Month, levels = rev(month.name), ordered = TRUE))

library(dplyr)
library(lubridate)
library(zoo)
library(plotly)

# Convert the Date for 2019 data from character to Date
waterkeeper_data_2019$Date <- as.Date(waterkeeper_data_2019$Date, format = "%m/%d/%Y")

# Aggregating data by week for 2019 data
waterkeeper_data_2019_aggregated <- waterkeeper_data_2019 %>%
  group_by(week = floor_date(Date, "week")) %>%
  summarise(
    avg_turbidity = mean(turbidity, na.rm = TRUE),
    avg_pH = mean(pH, na.rm = TRUE)
  ) %>%
  ungroup()

# Apply smoothing to aggregated data (5-period moving average)
waterkeeper_data_2019_aggregated$smooth_turbidity <- rollmean(waterkeeper_data_2019_aggregated$avg_turbidity, k = 5, fill = NA)
waterkeeper_data_2019_aggregated$smooth_pH <- rollmean(waterkeeper_data_2019_aggregated$avg_pH, k = 5, fill = NA)

#2019 data with separate axes
p_2019 <- plot_ly() %>%
  add_trace(
    data = waterkeeper_data_2019_aggregated,
    x = ~week,  # Using aggregated week
    y = ~smooth_turbidity,
    type = 'scatter',
    mode = 'lines',
    line = list(width = 2),
    name = 'Smoothed Turbidity',
    yaxis = "y1"  # Link to the first y-axis (for turbidity)
  ) %>%
  add_trace(
    data = waterkeeper_data_2019_aggregated,
    x = ~week,  # Using aggregated week
    y = ~smooth_pH,
    type = 'scatter',
    mode = 'lines',
    line = list(width = 2, dash = 'dot'),
    name = 'Smoothed pH',
    yaxis = "y2"  # Link to the second y-axis (for pH)
  ) %>%
  layout(
    title = 'Smoothed Turbidity and pH in 2019 (Aggregated by Week)',
    xaxis = list(title = 'Date'),
    yaxis = list(
      title = 'Turbidity (NTUs)',  # Label for the first y-axis (Turbidity)
      side = 'left'
    ),
    yaxis2 = list(
      title = 'pH',  # Label for the second y-axis (pH)
      side = 'right',
      overlaying = 'y',  # Overlay this y-axis on the same plot
      rangemode = 'tozero'  # Ensure pH axis starts at zero (optional)
    ),
    legend = list(title = list(text = 'Legend'))
  )

p_2019
```
```{r}
library(ggplot2)
library(sf)
library(readxl)
library(tidyverse)
waterkeeper_data_2023 <- read_excel("data/2023_Riverwatch_Data.xlsx")
#Convert the data into a spatial object
waterkeeper_data_2023_sf <- st_as_sf(waterkeeper_data_2023, coords = c("longitude", "latitude"), crs = 4326)

waterkeeper_data_2023_sf$longitude <- waterkeeper_data_2023$longitude 
waterkeeper_data_2023_sf$latitude <- waterkeeper_data_2023$latitude

waterkeeper_data_2023_st <- waterkeeper_data_2023_sf %>%
  mutate(
    Month = format(collection_date, "%B"),  # Extract month names
    Month = factor(Month, levels = rev(month.name), ordered = TRUE)  # Reverse the month order
  )

library(dplyr)
library(lubridate)
library(zoo)
library(plotly)

# Rename columns in the 2023 dataset first to avoid confusion
colnames(waterkeeper_data_2023)[colnames(waterkeeper_data_2023) == "Turbidity [p:2922]"] <- "turbidity"
colnames(waterkeeper_data_2023)[colnames(waterkeeper_data_2023) == "pH [p:2923]"] <- "pH"

# Convert collection_date to Date format for 2023 data (remove time portion)
waterkeeper_data_2023$collection_date <- as.Date(waterkeeper_data_2023$collection_date)

# Filter out rows with missing values for turbidity or pH
waterkeeper_data_2023 <- waterkeeper_data_2023 %>%
  filter(!is.na(pH) & !is.na(turbidity))

# Aggregating data by week for 2023 data
waterkeeper_data_2023_aggregated <- waterkeeper_data_2023 %>%
  group_by(week = floor_date(collection_date, "week")) %>%
  summarise(
    avg_turbidity = mean(turbidity, na.rm = TRUE),
    avg_pH = mean(pH, na.rm = TRUE)
  ) %>%
  ungroup()

# Apply smoothing to aggregated data (5-period moving average)
waterkeeper_data_2023_aggregated$smooth_turbidity <- rollmean(waterkeeper_data_2023_aggregated$avg_turbidity, k = 5, fill = NA)
waterkeeper_data_2023_aggregated$smooth_pH <- rollmean(waterkeeper_data_2023_aggregated$avg_pH, k = 5, fill = NA)

#2023 data with seperate axes
p_2023 <- plot_ly() %>%
  add_trace(
    data = waterkeeper_data_2023_aggregated,
    x = ~week,  # Using aggregated week
    y = ~smooth_turbidity,
    type = 'scatter',
    mode = 'lines',
    line = list(width = 2),
    name = 'Smoothed Turbidity',
    yaxis = "y1"  # Link to the first y-axis
  ) %>%
  add_trace(
    data = waterkeeper_data_2023_aggregated,
    x = ~week,  # Using aggregated week
    y = ~smooth_pH,
    type = 'scatter',
    mode = 'lines',
    line = list(width = 2, dash = 'dot'),
    name = 'Smoothed pH',
    yaxis = "y2"  # Link to the second y-axis
  ) %>%
  layout(
    title = 'Smoothed Turbidity and pH in 2023 (Aggregated by Week)',
    xaxis = list(title = 'Date'),
    yaxis = list(
      title = 'Turbidity (NTUs)',  # Label for the first y-axis
      side = 'left'
    ),
    yaxis2 = list(
      title = 'pH',  # Label for the second y-axis
      side = 'right',
      overlaying = 'y',  # Overlay this y-axis on the same plot
      rangemode = 'tozero'  # Make sure the pH axis starts at zero
    ),
    legend = list(title = list(text = 'Legend'))
  )

p_2023
```


# Conclusions

As aforementioned in my introduction, turbidity levels should not exceed 1 NTUs in order to be considered healthy enough to drink. However, according to the Buffalo Niagara Waterkeeper's 2021 Water Quality Report, turbidity levels should not exceed 5 NTUs. Despite these standards, turbidity levels in both years did not meet them. In 2019, turbidity levels ranged from 6.8 to 9.3, while ranging from 5.7 to 13.2 in 2023. The higher turbidity levels are generally seen in the summer months. This could be due to a number of reasons. The most likely reason for the increase in the summer months is increased algal blooms. As the U.S. Environmental Protection Agency's (EPA) factsheet on water quality parameters has noted, warmer temperatures and increased sunlight contribute to the growth of algae, especially in freshwater bodies (2021). This leads to the increased cloudiness of the water. Other factors that may contribute to increased turbidity include increased soil and sediment runoff from rainfall, reduced flow in water bodies due to higher evaporation rates, and human activity. 

For healthy freshwater systems, pH should range between 6.5 to 8.5. According to my analysis, the pH range hovers around 8. This indicates a slightly alkaline environment, but within a healthy range. This is most likely due to New York's limestone deposits. This also suggests the local waterways are less affected by any potential acidification as stated by the EPA (2021).

Overall, it seems as though my analysis proved my hypothesis incorrect. While the pH remained relatively consistent throughout both years, turbidity generally increased in 2023. This implies that human activity increased back to pre-COVID levels, or perhaps some significant environmental event occurred to reverse any potential effects of the lockdown. For future studies, I would compare the 2019 data with 2020 data to see if there are any decreased effects. I would also compare other values, such as total dissolved solids and oxygen, conductivity, temperature, and presence of E. coli. These are other factors that are standards of measurements of water quality data. I could further pursue changes depending on the time of year, effects of major storm events, different land use types, and nutrient levels present for biological organisms. 

# References

(2021, July). pH [Review of pH]. U.S. Environmental Protection Agency. https://www.epa.gov/system/files/documents/2021-07/parameter-factsheet_ph.pdf

(2021, July). Turbidity [Review of Turbidity]. U.S. Environmental Protection Agency. https://www.epa.gov/system/files/documents/2021-07/parameter-factsheet_turbidity.pdf

GeeksforGeeks. (2024, June 3). zoo package in R. GeeksforGeeks; GeeksforGeeks. https://www.geeksforgeeks.org/zoo-package-in-r/

Getting Started with Plotly in R [Review of Getting Started with Plotly in R]. Plotly Graphing Libraries. https://plotly.com/r/getting-started/#:~:text=plotly%20is%20an%20R%20package,js%20.

ggplot2: Elegant Graphics for Data Analysis (3e) - 1  Introduction. (n.d.). Ggplot2-Book.org. https://ggplot2-book.org/introduction

Ggpubr in R. (2023, January 26). GeeksforGeeks. https://www.geeksforgeeks.org/ggpubr-in-r/

Hadley Wickham. (2017). Easily Install and Load the “Tidyverse” [R package tidyverse version 1.2.1]. R-Project.org. https://cran.r-project.org/package=tidyverse

Hahn, N. (n.d.). 6 leaflet | Making Maps with R. In bookdown.org. https://bookdown.org/nicohahn/making_maps_with_r5/docs/leaflet.html

Hem, J. D. (2001). Study and Interpretation of the Chemical Characteristics of Natural Water Third Edition. https://pubs.usgs.gov/wsp/wsp2254/pdf/wsp2254a.pdf

How To Use Readxl Package To Read Data In R. (2022, June 24). GeeksforGeeks. https://www.geeksforgeeks.org/how-to-use-readxl-package-to-read-data-in-r/

(n.d.). Buffalo Niagara Waterkeeper [Review of Buffalo Niagara Waterkeeper]. Buffalo Niagara Waterkeeper. Retrieved December 10, 2024, from https://bnwaterkeeper.org/

Norris, R. H., & Thoms, M. C. (1999). What is river health? Freshwater Biology, 41(2), 197–209. https://doi.org/10.1046/j.1365-2427.1999.00425.x

Phillips, N. D. (2024, November 11). 10.4 dplyr | YaRrr! The Pirate’s Guide to R. Bookdown.org. https://bookdown.org/ndphillips/YaRrr/dplyr.html

Spatial Data Science - 7  Introduction to sf and stars. (2016). R-Spatial.org. https://r-spatial.org/book/07-Introsf.html

WATER QUALITY REPORT. (n.d.). https://bnwaterkeeper.org/wp-content/uploads/2022/01/2021-water-quality-report_FINAL_web.pdf

Working with Dates and Times in R Using the lubridate Package | UVA Library. (n.d.). Library.virginia.edu. https://library.virginia.edu/data/articles/working-with-dates-and-time-in-r-using-the-lubridate-package

World Health Organization. (2017, February 14). Water quality and health: Review of turbidity. Www.who.int. https://www.who.int/publications/i/item/WHO-FWC-WSH-17.01
