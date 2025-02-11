# Global Data Analysis Project
This project involves analyzing and visualizing various global datasets, including CO2 emissions, GDP, population, and fertility rates. The project includes data cleaning, adjustments, and generating insightful visualizations such as maps and histograms.

## Table of Contents
- [Project Overview](#project-overview)
- [Tools](#tools)
- [Data Preparation](#data-preparation)
- [Exploratory Data Analysis (EDA)](#exploratory-data-analysis-eda)
- [Data Analysis](#data-analysis)
- [Results/Findings](#resultsfindings)
- [Recommendations](#recommendations)
- [Limitations](#limitations)
- [References](#references)

---

### project overview 

This project analyzes global data for the year 2023, including details about country names, populations, CO2 emissions, GDP, and fertility rates. The aim is to identify patterns in CO2 emissions, GDP per capita, and fertility rates, as well as to highlight the top countries in these metrics. This analysis demonstrates skills in data cleaning, manipulation, visualization, and statistical analysis.

---
![Rplot](https://github.com/NeoSphereAnalytics/Global-CO2-Emissions-Analysis/assets/174109528/d637c910-501d-4ea6-815d-c5c2ec8f2ef5)

![Rplot05](https://github.com/NeoSphereAnalytics/Global-CO2-Emissions-Analysis/assets/174109528/7b96bb60-f1f5-45af-b661-4c5acdcda3fd)

![Rplot01](https://github.com/Ibrahim-Sereer/Global-CO2-Emissions-Analysis/assets/174109528/b56ab862-3caf-4f1b-a0ce-036bbc6ab285)

---

### Tools

- R - Data analysis and visualization [download here](https://www.r-project.org/)


---

### Data preperation 

Data Cleaning and Preparation Phase:
1. Adjusted country names for consistency.
2. Filled missing population values with the average population.
3. Calculated CO2 emissions per capita.
4. Merged the dataset with world map data.

---

### Exploratory Data Analysis (EDA):

Total Aggregates:
- What is the total number of countries recorded in the dataset?
- What is the distribution of CO2 emissions per capita?
- What are the top 10 countries with the highest CO2 emissions per capita?
  
Country-specific Insights:
- Which countries have the highest CO2 emissions per capita?
- What is the relationship between GDP per capita and fertility rates?

---

### Data analysis

```R
# Install necessary packages 
install.packages("ggplot2")   
install.packages("tidyverse") 
install.packages("cowplot")

# Load libraries
library(ggplot2)
library(tidyverse)
library(cowplot)
library(dplyr)

# Load the data
data <- read_csv("C:/Users/Owner/Downloads/archive (1)/world-data-2023.csv")

# Inspect the data to check for any potential issues
glimpse(data)
summary(data)

# Clean data: Adjust country names
data <- data %>%
  mutate(Country = case_when(
    Country == "Republic of China" ~ "China",
    Country == "United States of America" ~ "United States",
    Country == "Russian Federation" ~ "Russia",
    Country == "Korea, Republic of" ~ "South Korea",
    TRUE ~ Country
  ))

# Fill missing population values with a placeholder (e.g., average population)
average_population <- mean(data$Population, na.rm = TRUE)
data <- data %>%
  mutate(Population = ifelse(is.na(Population), average_population, Population))

# Calculate CO2 emissions per capita
data <- data %>%
  mutate(`Co2-Emissions per Capita` = `Co2-Emissions` / Population)

# Load world map data
mapdata <- map_data("world")

# Check for any mismatches and correct discrepancies
data <- data %>%
  mutate(Country = case_when(
    Country == "United States" ~ "USA",
    TRUE ~ Country
  ))

# Print unique country names in mapdata and data for verification
unique(mapdata$region) %>% sort()
unique(data$Country) %>% sort()

# Merge data with map data
mapdata <- left_join(mapdata, data, by = c("region" = "Country"))

# Check for missing values after merging
missing_mapdata <- mapdata %>% filter(is.na(`Co2-Emissions`))
print(missing_mapdata$region)

# Filter out rows with missing CO2 emissions data
mapdata_filtered <- mapdata %>% filter(!is.na(`Co2-Emissions`))

# Verify calculations
summary(mapdata_filtered)

# Create categories for CO2 emissions
mapdata_filtered <- mapdata_filtered %>%
  mutate(Co2_Category = cut(`Co2-Emissions per Capita`, 
                            breaks = quantile(`Co2-Emissions per Capita`, probs = seq(0, 1, by = 0.25), na.rm = TRUE), 
                            labels = c("Low", "Medium", "High", "Very High"),
                            include.lowest = TRUE))

# Create the map with categories
map <- ggplot(mapdata_filtered, aes(x = long, y = lat, group = group)) +
  geom_polygon(aes(fill = Co2_Category), color = "black") +
  scale_fill_manual(name = "CO2 Emissions per Capita", 
                    values = c("Low" = "yellow", "Medium" = "orange", "High" = "red", "Very High" = "darkred"), 
                    na.value = "grey50") +
  theme_void() +
  theme(
    legend.position = "bottom",
    plot.title = element_text(size = 15, face = "bold")
  ) +
  ggtitle("Global CO2 Emissions per Capita")

# Display the map
print(map)

# Sort the data by CO2 emissions and select the top 10 countries
top10_countries <- data %>%
  arrange(desc(`Co2-Emissions per Capita`)) %>%
  head(10)

# Create the histogram
histogram <- ggplot(top10_countries, aes(x = reorder(Country, `Co2-Emissions per Capita`), y = `Co2-Emissions per Capita`)) +
  geom_bar(stat = "identity", fill = "blue") +
  geom_text(aes(label = round(`Co2-Emissions per Capita`, 4)), vjust = -0.5, color = "black", size = 3.5) +
  labs(title = "Top 10 Countries with Highest CO2 Emissions per Capita",
       x = "Country",
       y = "CO2 Emissions per Capita") +
  ylim(0, max(top10_countries$`Co2-Emissions per Capita`) * 1.2) + # Adjust y-axis limit
  theme_minimal() +
  theme(
    plot.title = element_text(size = 15, face = "bold"),
    axis.text.x = element_text(angle = 45, hjust = 1),
    axis.title.x = element_text(size = 12),
    axis.title.y = element_text(size = 12)
  )

# Display the histogram
print(histogram)

# Clean and prepare the data: handle any missing values and convert relevant columns to numeric
data <- data %>%
  mutate(GDP = as.numeric(gsub("[$,]", "", GDP)),
         Population = as.numeric(gsub(",", "", Population)),
         `Fertility Rate` = as.numeric(`Fertility Rate`)) %>%
  filter(!is.na(GDP) & !is.na(`Fertility Rate`) & !is.na(Population))

# Calculate GDP per capita
data <- data %>%
  mutate(GDP_per_capita = GDP / Population)

# Perform linear regression
model <- lm(`Fertility Rate` ~ GDP_per_capita, data = data)

# Summarize the model
summary(model)

# Visualize the relationship with country labels
ggplot(data, aes(x = GDP_per_capita, y = `Fertility Rate`)) +
  geom_point() +
  geom_smooth(method = "lm", col = "red") +
  geom_text(aes(label = Country), size = 3, hjust = 0.5, vjust = -0.5, check_overlap = TRUE) +
  scale_x_continuous(labels = scales::dollar) +
  scale_y_continuous(expand = c(0, 0)) +
  coord_cartesian(xlim = c(0, max(data$GDP_per_capita, na.rm = TRUE)), ylim = c(0, max(data$`Fertility Rate`, na.rm = TRUE))) +
  labs(title = "Relationship between GDP per Capita and Fertility Rate",
       x = "GDP per Capita (in USD)",
       y = "Fertility Rate") +
  theme_minimal() +
  theme(plot.title = element_text(hjust = 0.5, size = 14, face = "bold"),
        axis.title.x = element_text(size = 12),
        axis.title.y = element_text(size = 12))

```

---

### Results/Findings

Total Aggregates:
- The dataset contains information on 195 unique countries.
- CO2 emissions per capita vary significantly across countries, with some having very high emissions.
- There is a noticeable relationship between GDP per capita and fertility rates.
  
Country-specific Insights:
- The United States, China, Canada, Saudia Arabia, and Russia are among the countries with the highest CO2 emissions per capita.
- The top 10 countries with the highest CO2 emissions per capita include major industrial nations as well as some very small states with small populations.
- Countries with Higher GDP per capita tends to correlate with lower fertility rates.

---

### Recommendations

- Implement targeted policies to reduce CO2 emissions in countries with the highest per capita emissions.
- Promote the use of renewable energy sources to decrease reliance on fossil fuels.
- Encourage international cooperation to address global CO2 emissions effectively.
- Consider economic policies that address the relationship between GDP per capita and fertility rates.

---

### Limitations

- Data Quality: The dataset's quality and completeness may affect the accuracy of the findings, and there may be inconsistencies or gaps in the reported values.
- Temporal Scope: The dataset is limited to 2023 and may not capture trends over multiple years.
- External Factors: The analysis does not account for external factors such as economic conditions, policies, or natural events that may influence CO2 emissions or fertility rates.

---

### References

Dataset Source:
- Global Country Information Dataset 2023: Available on Kaggle from [here](https://www.kaggle.com/datasets/nelgiriyewithana/countries-of-the-world-2023)

  
R Documentation:
- R Core Team (2023). R: A language and environment for statistical computing. R Foundation for Statistical Computing, Vienna, Austria. Available from [here](https://www.R-project.org/)

