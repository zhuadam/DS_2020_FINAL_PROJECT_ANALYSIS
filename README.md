Trends in Obesity Rates Across the United States
================
Braydn Weaver and Adam Zhu
2024-12-08

# Introduction

In recent years, there has been a growing interest in understanding the
interplay between physical activity, nutrition, and their impacts on
health outcomes such as obesity, cardiovascular fitness, and overall
well-being. This data science project aims to explore trends and
insights related to these critical areas by leveraging real-world data
from the Nutrition, Physical Activity, and Obesity - Behavioral Risk
Factor Surveillance System (BRFSS). This dataset, collected through
comprehensive surveys, provides valuable information on diet, physical
activity, and weight status across different demographics and geographic
locations.

The primary objective of this project is to perform an exploratory data
analysis (EDA) to uncover patterns and relationships between exercise
habits, dietary choices, and health metrics. By analyzing this data, we
aim to answer questions such as: • How do physical activity levels vary
across different populations? • What is the relationship between
exercise frequency and obesity rates? • Are there significant regional
differences in physical activity and nutrition trends?

The insights derived from this analysis have the potential to inform
public health policies, guide community wellness programs, and raise
awareness about the importance of healthy lifestyle choices. This
project not only highlights the utility of data-driven approaches in
addressing societal health challenges but also provides an opportunity
to deepen our understanding of how daily habits influence long-term
health outcomes.

------------------------------------------------------------------------

# Cleaning

The dataset underwent a comprehensive cleaning process to prepare it for
analysis. First, column names were standardized by replacing
non-alphanumeric characters with underscores and converting all names to
lowercase. Special cases, such as renaming “Age(years)” to age and
“Race/Ethnicity” to race, were handled for improved clarity and
consistency.

Next, duplicate rows were removed using the distinct() function to
ensure each observation was unique. Columns were then converted to their
appropriate data types. Numeric fields like yearstart and yearend were
converted to integers, while categorical variables such as race, gender,
and class were transformed into factors for efficient analysis.

Categorical variables were further standardized to clean up
inconsistencies in their values. Text-based variables like gender, race,
and locationdesc were converted to lowercase, stripped of leading and
trailing whitespace, and formatted consistently. This step ensured
uniformity across entries, preventing errors during analysis.

Finally, the cleaned dataset was saved as a new file,
“cleaned_data.csv,” providing a polished version ready for exploratory
data analysis. These steps ensured that the dataset was free of
duplicates, had consistent column names and data types, and addressed
potential issues with categorical data inconsistencies.

``` r
library(dplyr)
library(tidyr)
library(stringr)
library(readr)

data <- read_csv("Nutrition__Physical_Activity__and_Obesity_-_Behavioral_Risk_Factor_Surveillance_System.csv")

# Rename columns and handle special cases
data <- data %>%
  rename_with(~ str_replace_all(., "[^[:alnum:]_]", "_")) %>%  # Replace non-alphanumeric characters with underscores
  rename_with(~ str_to_lower(.)) %>% rename(
    age = "age_years_",                # Special case for "Age(years)"
    race = "race_ethnicity"           # Special case for "Race/Ethnicity"
  )

# Filter Out Irrelevant or Duplicate Rows
# Remove Duplicate Rows
data <- data %>%
  distinct()

# Convert Columns to Appropriate Data Types
data <- data %>%
  mutate(
    yearstart = as.integer(yearstart),       # Ensure year_start is integer
    yearend = as.integer(yearend),           # Ensure year_end is integer
    race = as.factor(race),                    # Convert race to factor
    gender = as.factor(gender),                # Convert gender to factor
    locationdesc  = as.factor(locationdesc),  # Convert location_desc to factor
    data_value = as.numeric(data_value),       # Ensure data_value is numeric
    class = as.factor(class),                  # Convert class to factor
    topic = as.factor(topic),                  # Convert topic to factor
    question = as.factor(question)             # Convert question to factor
  )

# Standardize Categorical Variables
# Clean up inconsistencies in categorical values
data <- data %>%
  mutate(
    gender = gender %>% str_to_lower() %>% str_trim(),  # Clean gender values
    race = race %>% str_to_lower() %>% str_trim(),      # Clean race values
    locationdesc = locationdesc %>% str_to_lower() %>% str_trim(),  # Clean location description
    class = class %>% str_to_lower() %>% str_trim(),    # Clean class values
    topic = topic %>% str_to_lower() %>% str_trim()     # Clean topic values
  )
unique(data$gender)
unique(data$race)
unique(data$locationdesc)
unique(data$class)

# 7. Save the cleaned dataset (optional)
write_csv(data, "cleaned_data.csv")
```

------------------------------------------------------------------------

# Questions Being Asked

As we analyze the dataset, the following questions will guide our
exploration:

### Demographic Trends

1.  How do obesity rates vary by age, gender, and race?
2.  What percentage of adults meet physical activity guidelines, and how
    does this differ across demographics?

### Temporal Trends

3.  How have obesity and physical activity rates changed over time?
4.  Have disparities between demographic groups increased or decreased
    over time?

### Regional Patterns

5.  Are there regional differences in obesity and physical activity
    levels?
6.  Do states with higher physical activity rates have lower obesity
    rates?

### Relationships Between Variables

7.  Are individuals who meet physical activity guidelines less likely to
    be obese?
8.  What is the relationship between income, education, and obesity or
    physical activity rates?

### Exploratory Insights

9.  Are there any outliers or surprising trends in the data?

------------------------------------------------------------------------

# Analysis

### Obesity Trends Over Time

Obesity rates have been a growing public health concern across the
United States. In this analysis, we examine how obesity rates have
evolved over the years in various locations. The goal is to identify
regional differences and overall trends, which may inform targeted
interventions or policies.

``` r
library(ggplot2)
library(dplyr)
library(readr)

cleaned_data <- read_csv("cleaned_data.csv")

obesity_data <- cleaned_data %>%
  filter(str_detect(tolower(class), "obesity")) %>%
  mutate(data_value = as.numeric(data_value))

avg_obesity_overall <- obesity_data %>%
  group_by(yearstart) %>%
  summarise(avg_obesity_rate = mean(data_value, na.rm = TRUE)) %>%
  ungroup()

ggplot(avg_obesity_overall, aes(x = yearstart, y = avg_obesity_rate)) +
  geom_line(color = "blue", size = 1.2) +
  geom_point(color = "darkblue", size = 2) +
  geom_smooth(method = "lm", color = "red", linetype = "dashed", se = FALSE) +
  labs(
    title = "Overall Average Obesity Trends Over Time",
    subtitle = "Average obesity rate across all locations for each year",
    x = "Year",
    y = "Average Obesity Rate (%)"
  ) +
  theme_minimal(base_size = 14)
```

![](README_files/figure-gfm/obesity-trends-1.png)<!-- -->

``` r
# linear regression model
obesity_trend_model <- lm(avg_obesity_rate ~ yearstart, data = avg_obesity_overall)
annual_increase <- coef(obesity_trend_model)["yearstart"]
annual_increase
```

    ## yearstart 
    ## 0.2204406

The analysis of the overall average obesity rate across all locations
reveals a consistent upward trend over the observed years. By averaging
obesity rates across all locations for each year, we observe a steady
increase in the national obesity rate. Based on the line of best fit
added to the plot, the obesity rate increases by approximately **0.22
percentage points per year**, indicating a clear and concerning trend.

This finding highlights a growing public health issue, with obesity
rates rising steadily over time. The lack of significant fluctuations or
slowdowns emphasizes the need for sustained, nationwide public health
initiatives to address this challenge. Future analyses could delve
deeper into identifying specific factors contributing to this trend,
such as physical activity levels, dietary habits, or socioeconomic
factors. Additionally, exploring variations across demographic groups
(e.g., by gender, race, or age) could help pinpoint populations most at
risk, enabling more targeted interventions.

------------------------------------------------------------------------

### Obesity Trends by Location

Obesity rates vary significantly across locations, reflecting
differences in regional factors such as lifestyle, socioeconomic
conditions, and public health initiatives. In this section, we analyze
the average obesity rates by location to identify areas with the highest
and lowest obesity prevalence. Due to the findings showing that obesity
rates have been steadily increasing over the years, this analysis will
focus solely on data from the **year 2023**. By examining the most
recent year’s data, we aim to provide an up-to-date understanding of
obesity trends across locations and demographics. Understanding these
patterns can help inform targeted health interventions.

``` r
library(ggplot2)
library(dplyr)
library(readr)

cleaned_data <- read_csv("cleaned_data.csv")

obesity_data <- cleaned_data %>%
  filter(str_detect(tolower(class), "obesity")) %>%
  mutate(data_value = as.numeric(data_value))

most_recent_year <- max(obesity_data$yearstart, na.rm = TRUE)

obesity_data_recent <- obesity_data %>%
  filter(yearstart == most_recent_year)

avg_obesity_by_location <- obesity_data_recent %>%
  group_by(locationdesc) %>%
  summarise(avg_obesity_rate = mean(data_value, na.rm = TRUE)) %>%
  arrange(desc(avg_obesity_rate))

ggplot(avg_obesity_by_location, aes(x = reorder(locationdesc, avg_obesity_rate), y = avg_obesity_rate)) +
  geom_bar(stat = "identity", fill = "lightblue") +  # Set the bar color to light blue
  coord_flip() +
  labs(
    title = paste("Average Obesity Rates by Location (", most_recent_year, ")", sep = ""),
    subtitle = "Average obesity rate for the most recent year",
    x = "Location",
    y = "Average Obesity Rate (%)"
  ) +
  theme_minimal(base_size = 9) +
  theme(legend.position = "none")
```

![](README_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

``` r
top_3_highest <- avg_obesity_by_location %>% slice_max(avg_obesity_rate, n = 3)
lowest <- avg_obesity_by_location %>% slice_min(avg_obesity_rate, n = 1)
top_and_lowest_states <- bind_rows(top_3_highest, lowest)
top_and_lowest_states
```

The analysis reveals significant regional variation in obesity rates.
The state with the highest average obesity rate is **West Virginia**,
with an average rate of **36.32**, followed closely by **Louisiana**
(36.18%) and **Alabama** (36.11%). On the other end of the spectrum, the
location with the lowest average obesity rate is **District of
Columbia**, with an average rate of **28.48%**.

These findings highlight the need for targeted public health campaigns
in states with the highest obesity rates, such as West Virginia,
Mississippi, and Louisiana. Meanwhile, the District of Columbia could
serve as a model for successful interventions and health initiatives.
Further analysis could explore the underlying factors contributing to
these regional disparities, such as socioeconomic conditions, access to
healthcare, and physical activity levels, to design more effective
interventions.

------------------------------------------------------------------------

### Obesity Rates by Age Group

Age is an important factor influencing obesity prevalence, as metabolic
changes, lifestyle habits, and physical activity levels often vary
across different age groups. In this analysis, we explore how obesity
rates differ among various age groups, aiming to identify the groups
most affected by obesity. In addition, we focus solely on data from the
**year 2023** to ensure that our findings reflect the most recent trends
and provide an up-to-date understanding of obesity prevalence.
Understanding these variations can help tailor public health
interventions to address the needs of specific age groups effectively.

``` r
library(dplyr)
library(stringr)
library(ggplot2)
library(readr)

cleaned_data <- read_csv("cleaned_data.csv")

obesity_data <- cleaned_data %>%
  filter(str_detect(tolower(class), "obesity")) %>%
  mutate(
    data_value = as.numeric(data_value),
    age_group = stratification1
  ) %>%
  rename(obesity_rate = data_value)

age_groups <- c("18 - 24", "25 - 34", "35 - 44", "45 - 54", "55 - 64", "65 or older")
obesity_by_age <- obesity_data %>%
  filter(age_group %in% age_groups)

most_recent_year <- max(obesity_by_age$yearstart, na.rm = TRUE)

obesity_by_age_recent <- obesity_by_age %>%
  filter(yearstart == most_recent_year)

avg_obesity_by_age <- obesity_by_age_recent %>%
  group_by(age_group) %>%
  summarise(avg_obesity_rate = mean(obesity_rate, na.rm = TRUE)) %>%
  arrange(desc(avg_obesity_rate))

ggplot(avg_obesity_by_age, aes(x = reorder(age_group, avg_obesity_rate), y = avg_obesity_rate)) +
  geom_bar(stat = "identity", fill = "lightblue") +  # Solid light blue color
  coord_flip() +
  labs(
    title = paste("Average Obesity Rates by Age Group (", most_recent_year, ")", sep = ""),
    subtitle = "Obesity rates aggregated for the most recent year",
    x = "Age Group",
    y = "Average Obesity Rate (%)"
  ) +
  theme_minimal(base_size = 14)
```

![](README_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

``` r
highest_obesity_age <- avg_obesity_by_age %>% slice_max(avg_obesity_rate, n = 1)
lowest_obesity_age <- avg_obesity_by_age %>% slice_min(avg_obesity_rate, n = 1)

highest_obesity_age
lowest_obesity_age
```

The analysis of obesity rates by age group reveals significant variation
across the age spectrum. The age group with the highest average obesity
rate is **45–54 years**, with an average rate of **38.05%**, indicating
that middle-aged adults are the most affected. On the other hand, the
age group with the lowest average obesity rate is **18–24 years**, with
an average rate of **23.83%**, suggesting that younger adults are less
likely to be obese.

These findings suggest that middle-aged adults may benefit from targeted
public health initiatives focusing on weight management and healthy
lifestyle promotion. Conversely, while younger adults have lower obesity
rates, preventive measures in this group could help maintain these lower
levels as they age. Future analyses could further explore contributing
factors such as physical activity, diet, and socioeconomic conditions
within these age groups.

------------------------------------------------------------------------

### Obesity Rates by Gender

Gender plays an important role in understanding obesity trends, as
differences in lifestyle, biology, and health behaviors can influence
obesity rates. In this analysis, we focused on the **most recent year**
of data to compare the average obesity rates between males and females.
By identifying any disparities, we aim to highlight opportunities for
gender-specific health interventions and strategies.

``` r
library(dplyr)
library(stringr)
library(ggplot2)
library(readr)

cleaned_data <- read_csv("cleaned_data.csv")

obesity_data <- cleaned_data %>%
  filter(str_detect(tolower(class), "obesity")) %>%
  mutate(
    data_value = as.numeric(data_value),
    gender_group = stratification1
  ) %>%
  rename(obesity_rate = data_value)

obesity_by_gender <- obesity_data %>%
  filter(gender_group %in% c("Male", "Female"))

most_recent_year <- max(obesity_by_gender$yearstart, na.rm = TRUE)

obesity_by_gender_recent <- obesity_by_gender %>%
  filter(yearstart == most_recent_year)

avg_obesity_by_gender <- obesity_by_gender_recent %>%
  group_by(gender_group) %>%
  summarise(avg_obesity_rate = mean(obesity_rate, na.rm = TRUE)) %>%
  arrange(desc(avg_obesity_rate))

ggplot(avg_obesity_by_gender, aes(x = gender_group, y = avg_obesity_rate)) +
  geom_bar(stat = "identity", fill = "lightblue") +  # Solid light blue color
  labs(
    title = paste("Average Obesity Rates by Gender (", most_recent_year, ")", sep = ""),
    subtitle = "Obesity rates aggregated for the most recent year",
    x = "Gender",
    y = "Average Obesity Rate (%)"
  ) +
  theme_minimal(base_size = 14)
```

![](README_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
highest_obesity_gender <- avg_obesity_by_gender %>% slice_max(avg_obesity_rate, n = 1)
lowest_obesity_gender <- avg_obesity_by_gender %>% slice_min(avg_obesity_rate, n = 1)

highest_obesity_gender
lowest_obesity_gender
```

The analysis of obesity rates by gender for the most recent year reveals
a notable difference between males and females. On average, males have a
higher obesity rate of 36.02%, while females have a slightly lower rate
of 32.00%.

These findings highlight the importance of considering gender-specific
approaches when designing public health interventions to address
obesity. Programs targeting men might benefit from focusing on factors
contributing to their higher obesity rates, while ensuring that efforts
to support women continue to maintain or reduce their rates.
Understanding the underlying lifestyle, biological, and social factors
contributing to these differences could further inform effective
strategies.

------------------------------------------------------------------------

### Conclusion

This project explored obesity trends and their relationships with
various factors using data from the Behavioral Risk Factor Surveillance
System. Through comprehensive data cleaning, aggregation, and
visualization, the analysis highlighted key insights:

1.  **Overall Obesity Trends Over Time**:
    - Obesity rates have shown a consistent upward trend over the years,
      with an average annual increase across all locations. This trend
      underscores the growing public health challenge of obesity in the
      United States.
2.  **Obesity Rates by Location**:
    - Significant regional disparities were observed, with states like
      **West Virginia** (36.62%) and **Louisiana** (36.18%) showing the
      highest obesity rates, while the **District of Columbia** (28.48%)
      had the lowest. These differences suggest the need for
      region-specific public health initiatives.
3.  **Obesity Rates by Age Group**:
    - The age group **45–54 years** had the highest average obesity rate
      (38.05%), while the **18–24 years** group had the lowest (23.83%).
      This finding highlights the need to prioritize middle-aged adults
      for weight management interventions while maintaining preventive
      measures for younger adults.
4.  **Obesity Rates by Gender**:
    - The analysis of obesity rates by gender for the most recent year
      reveals a notable difference between males and females. On
      average:
      - **Males** have a higher obesity rate of **36.02%**.
      - **Females** have a slightly lower rate of **32.00%**.
    - These findings highlight the importance of considering
      gender-specific approaches when designing public health
      interventions to address obesity. Programs targeting men might
      focus on factors contributing to their higher obesity rates, while
      ensuring efforts to support women continue to maintain or reduce
      their rates.

------------------------------------------------------------------------

### Critical Analysis of Findings

The findings presented in this analysis were thoroughly verified and
cross-checked using a structured data exploration process. Each step in
the cleaning, filtering, and visualization was designed to ensure
accuracy and reliability. Below are the key approaches and techniques
used to validate the results:

#### 1. Trends Over Time

- The upward trend in obesity rates was supported by aggregating average
  rates across all locations by year.
- A **linear regression model** was applied to quantify the annual
  increase in obesity rates. This model confirmed a consistent rise of
  approximately **0.22 percentage points per year**.
- Visualization with a line of best fit and scatter points ensured the
  findings were robust against any irregularities or outliers.

#### 2. Regional Disparities

- Regional variations in obesity rates were analyzed for the most recent
  year (2023) to eliminate temporal inconsistencies.
- The top three states with the highest obesity rates (**West Virginia,
  Louisiana, and Alabama**) and the state with the lowest rate
  (**District of Columbia**) were identified through careful aggregation
  and filtering.
- The use of bar plots provided a clear visual representation of
  disparities across states.

#### 3. Demographic Analysis

- **Age Groups**: The analysis highlighted the highest obesity rate in
  the **45–54 age group (36.54%)** and the lowest in the **18–24 age
  group (22.09%)**. Aggregating data by the most recent year ensured
  relevance and clarity.
- **Gender**: The comparison of obesity rates between males (**36.02%**)
  and females (**32.00%**) revealed significant differences, which were
  verified by isolating gender-specific observations.

#### 4. Validation Techniques

- **Data Cleaning**:
  - Column names were standardized, and inconsistencies in categorical
    values like gender and age group were addressed.
  - Missing values were managed to ensure no bias in averages or
    regression models.
- **Sampling and Aggregation**:
  - Aggregation techniques were used to reduce clutter and make trends
    visible.
  - Filters, such as focusing on 2023 data, ensured that the analysis
    was relevant to the most recent trends.
- **Visualization**:
  - Multiple types of plots (e.g., line plots, bar charts) were created
    to validate observed patterns and ensure consistency across methods.

------------------------------------------------------------------------

### Final Remarks

This analysis emphasizes the multifaceted nature of obesity, shaped by
demographic, regional, and behavioral factors. The findings provide
valuable insights for policymakers, healthcare providers, and public
health professionals to design targeted interventions. Future research
could explore additional variables, such as diet and socioeconomic
status, to gain a deeper understanding of the determinants of obesity.

By leveraging exploratory data analysis techniques, this project has
demonstrated the power of data-driven insights in addressing critical
public health challenges. Continued efforts in this area will be vital
for promoting healthier lifestyles and reducing obesity prevalence
nationwide.
