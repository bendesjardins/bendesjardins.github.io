---
title: DANL Project
subtitle: "Data-Driven Mastery: Unlocking Business Potential"
author: 
  - Kohl Courtwright
  - Benjamin DesJardins
  - AJ Forte
  - Misfer Hamid
  - Shannon Pierce

toc: true
toc-title: "Table of Contents"
toc-depth: 2
number-sections: true

fig-width: 9

execute:
  code-tools: true
  message: false
  warning: false

from: markdown+emoji
---




# Introduction

Introduction to the NYC Rolling Sales Dataset

The NYC Rolling Sales dataset provides a comprehensive overview of real estate transactions within New York City. Compiled from the city's rolling sales data, this dataset encompasses a wide range of property sales, offering insights into the dynamic and diverse real estate market of one of the world's most iconic metropolitan areas.

Key Information:

Source: NYC Department of Finance
Coverage: The dataset includes information on sales transactions across various boroughs, capturing the intricacies of the city's real estate landscape.
Columns: Essential details such as property address, sale price, sale date, borough, tax class code, and other pertinent information are included.
Objective:
The primary objective of this dataset is to facilitate analysis and exploration of real estate trends, patterns, and market dynamics within New York City. Researchers, analysts, and enthusiasts can leverage this dataset to gain insights into property values, transaction volumes, and the distribution of sales across different tax class codes and boroughs.

Usage:
This dataset is valuable for a variety of purposes, including market research, trend analysis, and the identification of patterns that may influence property values. It provides a robust foundation for those interested in understanding the factors influencing real estate transactions in the diverse neighborhoods of New York City.

Accessing the Data:
The dataset is publicly accessible and can be obtained from the NYC Department of Finance. In this analysis, we utilize the dataset available at https://bendesjardins.github.io/nyc-rolling-sales.csv.

# Data

The data.frame `nyc_estate` is a set of data that is a record of every building or building unit (apartment, etc.) sold in the New York City property market over a 12-month period. It is available at <https://www.kaggle.com/datasets/new-york-city/nyc-property-sales> for free use.

## Summary Statistics


::: {.cell}

```{.r .cell-code}
nyc_estate <- read_csv("https://bendesjardins.github.io/nyc-rolling-sales.csv")
```
:::

::: {.cell-output-display}
`````{=html}
<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
  </script>
</div>
`````
:::


## Distribution of Key Data

### Sale Price


::: {.cell}

```{.r .cell-code}
clean_sale <- nyc_estate %>% 
  filter(!(`SALE PRICE` == "-"))

ggplot(data = clean_sale) +
  geom_histogram(aes(x = log(as.numeric(`SALE PRICE`))), bins = 200) +
  labs(x = "Log of `Sale Price`")
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-3-1.png){width=864}
:::
:::


After taking the log of `Sale Price`, we can see that there seems to be a pretty normal distrbution with a few outliers near the lower range of data.

### TAX CLASS AT TIME OF SALE


::: {.cell}

```{.r .cell-code}
ggplot(data = nyc_estate) +
  geom_bar(aes(x = `TAX CLASS AT TIME OF SALE`))
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-4-1.png){width=864}
:::
:::


`TAX CLASS AT TIME OF SALE` is a categorical variable taking on the values of `1`, `2`, and `4` with most of the variables being either a `1` or `2`.

### YEAR BUILT


::: {.cell}

```{.r .cell-code}
clean_year <- nyc_estate %>% 
  filter(!(`YEAR BUILT` == 0))

ggplot(data = clean_year) + 
  geom_histogram(aes(x = `YEAR BUILT`), bins = 100) + 
  scale_x_continuous(limits = c(1800, 2020))
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-5-1.png){width=864}
:::
:::


## Comparison with Sale Price

### Sale Price versus Tax Class At Time of Sale


::: {.cell}

```{.r .cell-code}
ggplot(data = clean_sale) +
  geom_histogram(aes(x = log(as.numeric(`SALE PRICE`)), 
                     color = `TAX CLASS AT TIME OF SALE`), bins = 100) +
  facet_wrap(. ~ `TAX CLASS AT TIME OF SALE`, scales = 'free') +
  labs(x = "Log of `SALE PRICE`")
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-6-1.png){width=864}
:::
:::


They all have some resemblance of a normal distribution, with those at a tax class of `2` having the most normal distribution. Those in the `4` class has the most skewed data with many outliers in proportion to the rest of those in `1` and `2`.

### Sale Price versus Year Built


::: {.cell}

```{.r .cell-code}
clean_year_sale <- clean_sale %>% 
  filter(!(`YEAR BUILT` == 0))

ggplot(data = clean_year_sale) +
  geom_point(aes(x = log(as.numeric(`SALE PRICE`)), y = `YEAR BUILT`), alpha = .02) +
  scale_y_continuous(limits = c(1900, 2020))
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-7-1.png){width=864}
:::

```{.r .cell-code}
cor(as.numeric(clean_year_sale$`SALE PRICE`), clean_year_sale$`YEAR BUILT`)
```

::: {.cell-output .cell-output-stdout}
```
[1] 0.006318972
```
:::
:::

As the figure shows, there is very little correlation between year built and sale price. If we take the correlation value between them, we get less than 0.01 which means there is essentially no correlation. 

### Tax Class at Time of Sale versus Year Built

::: {.cell}

```{.r .cell-code}
ggplot(data = clean_year) +
  geom_histogram(aes(x = `YEAR BUILT`), bins = 50) +
  facet_wrap(. ~ `TAX CLASS AT TIME OF SALE`) + 
  scale_x_continuous(limits = c(1900, 2020))
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-8-1.png){width=864}
:::
:::


## Analysis of Buildings

### Number of Buildings

::: {.cell}

```{.r .cell-code}
nyc_estate <- read_csv("https://bendesjardins.github.io/nyc-rolling-sales.csv")


num_buildings <- nrow(nyc_estate)

cat("The number of buildings is:", num_buildings, "\n")
```

::: {.cell-output .cell-output-stdout}
```
The number of buildings is: 84548 
```
:::
:::



### Number of Apartments Per Borough

1 = Manhattan
2 = Bronx
3 = Brooklyn
4 = Queens
5 = Staten Island

::: {.cell}

```{.r .cell-code}
apartments_per_borough <- nyc_estate %>%
  group_by(BOROUGH) %>%
  summarize(Number_of_Apartments = n())

print(apartments_per_borough)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 5 × 2
  BOROUGH Number_of_Apartments
    <dbl>                <int>
1       1                18306
2       2                 7049
3       3                24047
4       4                26736
5       5                 8410
```
:::

```{.r .cell-code}
ggplot(apartments_per_borough, aes(x = factor(BOROUGH), y = Number_of_Apartments, fill = factor(BOROUGH))) +
  geom_bar(stat = "identity", color = "black", alpha = 0.7) +
  labs(title = "Number of Apartments per Borough",
       x = "Borough",
       y = "Number of Apartments",
       fill = "Borough") +
  theme_minimal()
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-10-1.png){width=864}
:::
:::


### Buildings in the same Zipcodes


::: {.cell}

```{.r .cell-code}
buildings_per_zip <- nyc_estate %>%
  group_by(`ZIP CODE`) %>%
  summarize(Number_of_Buildings = n())

print(buildings_per_zip)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 186 × 2
   `ZIP CODE` Number_of_Buildings
        <dbl>               <int>
 1          0                 982
 2      10001                 204
 3      10002                 328
 4      10003                 812
 5      10004                  95
 6      10005                 199
 7      10006                 184
 8      10007                 313
 9      10009                 244
10      10010                 459
# ℹ 176 more rows
```
:::
:::



### Buildings in same Tax Class

::: {.cell}

```{.r .cell-code}
buildings_per_tax_class <- nyc_estate %>%
  group_by(`TAX CLASS AT PRESENT`) %>%
  summarize(Number_of_Buildings = n())

# Create a scatter plot for the number of buildings per tax class
ggplot(buildings_per_tax_class, aes(x = `TAX CLASS AT PRESENT`, y = Number_of_Buildings, color = `TAX CLASS AT PRESENT`)) +
  geom_point(size = 3) +
  labs(title = "Number of Buildings per Tax Class",
       x = "Tax Class",
       y = "Number of Buildings",
       color = "Tax Class") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 45, hjust = 1))
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-12-1.png){width=864}
:::
:::


## Commericial and Residential Units
 
### Number of Commercial Units 


::: {.cell}

```{.r .cell-code}
commercial_units <- nyc_estate %>%
  filter(`BUILDING CLASS CATEGORY` %in% c("21 OFFICE BUILDINGS"))

total_commercial_units <- sum(commercial_units$`TOTAL UNITS`)


cat("Total Commercial Units:", total_commercial_units, "\n")
```

::: {.cell-output .cell-output-stdout}
```
Total Commercial Units: 2305 
```
:::
:::



### Number of Residential Units



::: {.cell}

```{.r .cell-code}
residential_units <- nyc_estate %>%
  filter(`BUILDING CLASS CATEGORY` %in% c("01 ONE FAMILY DWELLINGS", "02 TWO FAMILY DWELLINGS", "10 COOPS - ELEVATOR APARTMENTS"))


total_residential_units <- sum(residential_units$`TOTAL UNITS`)

cat("Total Residential Units:", total_residential_units, "\n")
```

::: {.cell-output .cell-output-stdout}
```
Total Residential Units: 63776 
```
:::
:::

 
### Highest and Lowest sold Residential Units


::: {.cell}

```{.r .cell-code}
index_highest <- which.max(residential_units$`SALE PRICE`)

index_lowest <- which.min(residential_units$`SALE PRICE`)


highest_price_property <- residential_units[index_highest, ]
lowest_price_property <- residential_units[index_lowest, ]

cat("Residential Unit with Highest Sales Price:\n")
```

::: {.cell-output .cell-output-stdout}
```
Residential Unit with Highest Sales Price:
```
:::

```{.r .cell-code}
print(highest_price_property)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 1 × 22
   ...1 BOROUGH NEIGHBORHOOD BUILDING CLASS CATEG…¹ `TAX CLASS AT PRESENT` BLOCK
  <dbl>   <dbl> <chr>        <chr>                  <chr>                  <dbl>
1 11983       1 UPPER EAST … 10 COOPS - ELEVATOR A… 2                       1392
# ℹ abbreviated name: ¹​`BUILDING CLASS CATEGORY`
# ℹ 16 more variables: LOT <dbl>, `EASE-MENT` <lgl>,
#   `BUILDING CLASS AT PRESENT` <chr>, ADDRESS <chr>, `APARTMENT NUMBER` <chr>,
#   `ZIP CODE` <dbl>, `RESIDENTIAL UNITS` <dbl>, `COMMERCIAL UNITS` <dbl>,
#   `TOTAL UNITS` <dbl>, `LAND SQUARE FEET` <chr>, `GROSS SQUARE FEET` <chr>,
#   `YEAR BUILT` <dbl>, `TAX CLASS AT TIME OF SALE` <dbl>,
#   `BUILDING CLASS AT TIME OF SALE` <chr>, `SALE PRICE` <chr>, …
```
:::
:::

::: {.cell}

```{.r .cell-code}
cat("\nResidential Unit with Lowest Sales Price:\n")
```

::: {.cell-output .cell-output-stdout}
```

Residential Unit with Lowest Sales Price:
```
:::

```{.r .cell-code}
print(lowest_price_property)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 1 × 22
   ...1 BOROUGH NEIGHBORHOOD BUILDING CLASS CATEG…¹ `TAX CLASS AT PRESENT` BLOCK
  <dbl>   <dbl> <chr>        <chr>                  <chr>                  <dbl>
1     9       2 BATHGATE     01 ONE FAMILY DWELLIN… 1                       3048
# ℹ abbreviated name: ¹​`BUILDING CLASS CATEGORY`
# ℹ 16 more variables: LOT <dbl>, `EASE-MENT` <lgl>,
#   `BUILDING CLASS AT PRESENT` <chr>, ADDRESS <chr>, `APARTMENT NUMBER` <chr>,
#   `ZIP CODE` <dbl>, `RESIDENTIAL UNITS` <dbl>, `COMMERCIAL UNITS` <dbl>,
#   `TOTAL UNITS` <dbl>, `LAND SQUARE FEET` <chr>, `GROSS SQUARE FEET` <chr>,
#   `YEAR BUILT` <dbl>, `TAX CLASS AT TIME OF SALE` <dbl>,
#   `BUILDING CLASS AT TIME OF SALE` <chr>, `SALE PRICE` <chr>, …
```
:::
:::




### Highest and Lowest sold Commercial Units


::: {.cell}

```{.r .cell-code}
commercial_units <- nyc_estate %>%
  filter(`BUILDING CLASS CATEGORY` %in% c("21 OFFICE BUILDINGS"))


highest_price_property_commercial <- commercial_units[which.max(commercial_units$`SALE PRICE`), ]


lowest_price_property_commercial <- commercial_units[which.min(commercial_units$`SALE PRICE`), ]


cat("Commercial Unit with Highest Sales Price:\n")
```

::: {.cell-output .cell-output-stdout}
```
Commercial Unit with Highest Sales Price:
```
:::

```{.r .cell-code}
print(highest_price_property_commercial)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 1 × 22
   ...1 BOROUGH NEIGHBORHOOD BUILDING CLASS CATEG…¹ `TAX CLASS AT PRESENT` BLOCK
  <dbl>   <dbl> <chr>        <chr>                  <chr>                  <dbl>
1  7451       1 MIDTOWN CBD  21 OFFICE BUILDINGS    4                       1301
# ℹ abbreviated name: ¹​`BUILDING CLASS CATEGORY`
# ℹ 16 more variables: LOT <dbl>, `EASE-MENT` <lgl>,
#   `BUILDING CLASS AT PRESENT` <chr>, ADDRESS <chr>, `APARTMENT NUMBER` <chr>,
#   `ZIP CODE` <dbl>, `RESIDENTIAL UNITS` <dbl>, `COMMERCIAL UNITS` <dbl>,
#   `TOTAL UNITS` <dbl>, `LAND SQUARE FEET` <chr>, `GROSS SQUARE FEET` <chr>,
#   `YEAR BUILT` <dbl>, `TAX CLASS AT TIME OF SALE` <dbl>,
#   `BUILDING CLASS AT TIME OF SALE` <chr>, `SALE PRICE` <chr>, …
```
:::
:::

::: {.cell}

```{.r .cell-code}
cat("\nCommercial Unit with Lowest Sales Price:\n")
```

::: {.cell-output .cell-output-stdout}
```

Commercial Unit with Lowest Sales Price:
```
:::

```{.r .cell-code}
print(lowest_price_property_commercial)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 1 × 22
   ...1 BOROUGH NEIGHBORHOOD BUILDING CLASS CATEG…¹ `TAX CLASS AT PRESENT` BLOCK
  <dbl>   <dbl> <chr>        <chr>                  <chr>                  <dbl>
1   695       2 BEDFORD PAR… 21 OFFICE BUILDINGS    4                       3309
# ℹ abbreviated name: ¹​`BUILDING CLASS CATEGORY`
# ℹ 16 more variables: LOT <dbl>, `EASE-MENT` <lgl>,
#   `BUILDING CLASS AT PRESENT` <chr>, ADDRESS <chr>, `APARTMENT NUMBER` <chr>,
#   `ZIP CODE` <dbl>, `RESIDENTIAL UNITS` <dbl>, `COMMERCIAL UNITS` <dbl>,
#   `TOTAL UNITS` <dbl>, `LAND SQUARE FEET` <chr>, `GROSS SQUARE FEET` <chr>,
#   `YEAR BUILT` <dbl>, `TAX CLASS AT TIME OF SALE` <dbl>,
#   `BUILDING CLASS AT TIME OF SALE` <chr>, `SALE PRICE` <chr>, …
```
:::
:::



### Scatter plot of highest and lowest sold units


::: {.cell}

```{.r .cell-code}
ggplot(commercial_units, aes(x = `SALE PRICE`, y = `TOTAL UNITS`)) +
  geom_point(color = "skyblue", size = 3) +
  geom_point(data = highest_price_property_commercial, aes(x = `SALE PRICE`, y = `TOTAL UNITS`), color = "red", size = 3) +
  geom_point(data = lowest_price_property_commercial, aes(x = `SALE PRICE`, y = `TOTAL UNITS`), color = "green", size = 3) +
  labs(title = "Scatter Plot for Highest and Lowest Sold Commercial Units",
       x = "Sale Price",
       y = "Total Units") +
  theme_minimal()
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-19-1.png){width=864}
:::
:::


 
### Number of Buildings per Neighborhood


::: {.cell}

```{.r .cell-code}
buildings_per_neighborhood <- nyc_estate %>%
  group_by(`NEIGHBORHOOD`) %>%
  summarize(Number_of_Buildings = n())


print(buildings_per_neighborhood)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 254 × 2
   NEIGHBORHOOD         Number_of_Buildings
   <chr>                              <int>
 1 AIRPORT LA GUARDIA                     8
 2 ALPHABET CITY                        204
 3 ANNADALE                             198
 4 ARDEN HEIGHTS                        278
 5 ARROCHAR                              45
 6 ARROCHAR-SHORE ACRES                  33
 7 ARVERNE                              197
 8 ASTORIA                             1216
 9 BATH BEACH                           272
10 BATHGATE                              68
# ℹ 244 more rows
```
:::
:::



### Number of Buildings most and lowest in a block

::: {.cell}

```{.r .cell-code}
buildings_per_block <- nyc_estate %>%
  group_by(BLOCK) %>%
  summarize(Number_of_Buildings = n())


most_buildings_block <- buildings_per_block %>%
  filter(Number_of_Buildings == max(Number_of_Buildings))


fewest_buildings_block <- buildings_per_block %>%
  filter(Number_of_Buildings == min(Number_of_Buildings))


cat("Block with Most Buildings:\n")
```

::: {.cell-output .cell-output-stdout}
```
Block with Most Buildings:
```
:::

```{.r .cell-code}
print(most_buildings_block)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 1 × 2
  BLOCK Number_of_Buildings
  <dbl>               <int>
1  5066                 404
```
:::

```{.r .cell-code}
cat("Block with Fewest Buildings:\n")
```

::: {.cell-output .cell-output-stdout}
```
Block with Fewest Buildings:
```
:::

```{.r .cell-code}
print(fewest_buildings_block)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 1,983 × 2
   BLOCK Number_of_Buildings
   <dbl>               <int>
 1     5                   1
 2    19                   1
 3    22                   1
 4    48                   1
 5    71                   1
 6    83                   1
 7    88                   1
 8   112                   1
 9   113                   1
10   126                   1
# ℹ 1,973 more rows
```
:::

```{.r .cell-code}
ggplot(buildings_per_block, aes(x = Number_of_Buildings)) +
  geom_freqpoly(binwidth = 1, fill = "skyblue", color = "black", alpha = 0.7) +
  labs(title = "Frequency Polygon of Number of Buildings per Block",
       x = "Number of Buildings",
       y = "Frequency") +
  theme_minimal()
```

::: {.cell-output-display}
![](project_files/figure-html/unnamed-chunk-21-1.png){width=864}
:::
:::


# Analysis

Utilizing the `NYC Rolling Sales` data set to study the relationships between the variables, we were able to determine predictability between all the variables. Some of the things taken into consideration was `SALE PRICE`, `TAX CLASS AT TIME OF SALE`, `NUMBER OF BUILDINGS`, `BOROUGH`, and more. 

## Findings Regarding Sale Price
`SALE PRICE` had a normal distribution with some degree of outliers either at a price of 0 or something very low. This can be contributed to pieces of land being given away at a low price or other some either reason than just to be sold. `SALE PRICE` was compared primarily with `TAX CLASS AT TIME OF SALE` and `YEAR BUILT` to determine any correlations. When it came it `TAX CLASS AT TIME OF SALE`, `SALE PRICE` kept a similar normal distribution to the original data, all of it falling into the same general range. However, buildings with a tax class of `4` had the highest proportion of outliers compared to the rest of the data, as the figure in 2.3.1 shows. This implies that aside from having a higher chance of being a very low sale given that the building was in the tax class of 4, no assumptions can be made about the `SALE PRICE` with a varying tax class.

Another variable tested was `YEAR BUILT` and as demonstrated by calculating the correlation coefficient and the graph in 2.3.2, there is essentially no correlation between the two variables. 

## Findings Regarding Number of Buildings

Another topic assessed was how the distribution of buildings related to other variables. The first thing recognized was the distribution of buildings in comparison to which borough they were located. The most common location was in Queens (4), and the least number of apartments sold was in the Bronx. Another variable studied in relation to number of buildings was `TAX CLASS`. The most common tax classes were 1 and 2, in a range of 30000 to 40000 buildings (Out of 80000 total buildings). There were a select few in the 4 category, and some of the sub categories had anywhere from 1000 to 3000 buildings. Three was the least common tax class, as demonstrated by some of the previous graphs. 

## Findings Regarding Commercial versus Residential Units
One of the last topics considered was whether or not buildings were commercial/residential units. This was done by filtering the `BUILDING CLASS CATEGORY`, where those that were listed as ``21 OFFICE BUILDINGS`` were classified was commercial, and those that were family dwellings/coops were marked as residential. Residential Units greatly outnumbered commercial units, with a ratio of 63776 : 2305 or about 55 : 2. Thus the findings regarding residential units have more data to back it up. Both the top ten buildings with the lowest `SALE PRICE` and highest `SALE PRICE` for both commercial and residential buildings were found to provide insight to what differences may come up. The figure in 2.5.5 showcases where these buildings land in comparison with the rest.

# Conclusion

The topics discussed and analyzed only covered a small portion of what the `nyc-rolling-sales.csv` data set has to offer. It was established that `SALE PRICE` had no relationship with some of the variables such as `YEAR BUILT` and no serious predictions could be made on what the sale price of buildings could be with any of the variables we looked at. Studying the distribution of buildings with variables like `TAX CLASS` also provided little insight on prediction. The last thing assessed was commercial versus residential units, where it was established the number of residential buildings greatly outclassed the number of commercial ones. In the future, finding ways to intertwine the categories assessed and study the connections between them would continue to provide further insight on New York City real estate listings. 