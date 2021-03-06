---
title: "Health Care Expenditure vs. Social Indicators: A lookup on how the former affects the latter"
author: "Thomas Green"
date: "27/02/2022"
output: rmarkdown::github_document
---

*Disclaimer: This document was generated with the sole purpose of serving as a portfolio Data Analysis project, not intended to structure and perform a real scientific study since I'm not an expert on the subject and neither consulted with experts to guide the study, serving only as a technical display.*

## Research Question and Methodology

This study is about the relationship between health care expenditure and socioeconomic indicators around the world, focusing on analyzing three aspects:

**A) Health care expenses over GDP (Gross Domestic Product) and per capita expenses (Public and Private).**

**B) Infant and adult mortality rates.**

**C) Life expectancy at birth.**

The analytical process started by understanding which countries spent the most and least proportionally per citizen, which populations spent the most and least on out-of-pocket private health care and complementary an overview of how much these expenditures grew between 2001-2019 (available data in time series). In this analysis, External expenditures in the countries were ignored since they represent foreign aids, occasional supports, and not the internal reality in these countries.

In sequence, the analysis on mortality rates for both infant and adults, generated a rank from greatest to smallest rates for each group, followed by an evaluation of what was the variation in these rates between 2001-2019. These new variables were complemented by the public and private expenditures variations at the same period, to compare with the evolution of such mortality rates.

The third aspect of life expectancy wrapped the analytical process, using the same approach of the second aspect by starting with a rank of countries at this indicator and then analyzing its evolution over time, comparing to the evolution of expenditures on health care.

At conclusion, a better understanding of how much the value spent by countries on health care reflects on an improvement in social indicators was generated, providing insights on what data highlighted about this expenditure vs. return ratio.

Obs.: Countries without the necessary records at each stage were ignored.

## Data Collection

For this study it was collected online data sets from the World Bank website. It provided the World Development Indicators (WDI) report, in which only data from 2001 to 2020 (last 20 years available) was considered for this analysis, though not all years had data for every country and non-available results were removed from the data frame during the preparation step.

To obtain the same data sets used, it's recommended that at the Series filter, only the 'Economic Policy and Debt', 'Health' and 'Social: health' items are selected in order to limit the scope of indicators and allow extraction from the website, while filtering only 10 years each time, since there is a records limit per query.

## Data Preparation

The first step was to load the necessary packages.

```{r, message = FALSE}
library("tidyverse")
```

Next, loading the individual data sets and unifying them into a single data frame with all the 20 years of data, through the column binding function with a slicing in the second data set, to ignore the duplicated columns.

```{r, message = FALSE}
ds1 <- read_csv("C:/*/World Development Indicators/WDI 2001-2010.csv")
ds2 <- read_csv("C:/*/World Development Indicators/WDI 2011-2020.csv")

data <- cbind(ds1, ds2[, 5:14])
```

Following, a column renaming step to standardize the names and fix years redundancy. Also, it's needed to pivot the yearly columns to rows since those are observations, not variables.

```{r}
colnames(data)[5:24] <- c(2001:2020)

data <- pivot_longer(data, cols = starts_with("2"),  names_to = "Year", values_to = "Result")
```

Finally, perform the final cleaning processes to change the blank results value pattern ('..') from the original data set, change the 'Result' column data type to numeric and remove both aggregate regional and NA records.

```{r}
options(scipen = 999) # Remove scientific notation display in results

data <- cbind(data[1:5], replace(data[6], data[6] == "..", ""))

df <- data.frame(data[1:5], Results = round(as.numeric(data$Result), digits = 2))

df <- subset(df, !df$Country.Code %in% c("CAF", "CSS", "LAO", "ZAF", "AFE", "AFW", "ARB", "CEB", "EAR", "EAS", "EAP", "TEA", "EMU", "ECS", "ECA", "TEC", "EUU", "FCS", "HPC", "HIC", "IBD", "IBT", "IDB", "IDX", "IDA", "LTE", "LCN", "LAC", "TLA", "LDC", "LMY", "LIC", "LMC", "MEA", "MNA", "TMN", "MIC", "NAC", "INX", "OED", "OSS", "PSS", "PST", "PRE", "SST", "SAS", "TSA", "SSF", "SSA", "TSS", "UMC", "WLD"))

df <- drop_na(df)
```

## Data Analysis

The analysis begins with the health care expenditure per capita indicator (in US$), considering both public and private systems, to verify which countries spent the most and the least on each in 2019, limiting results for plotting to the top 10. To complement this information, a rank was generated and aggregated to the tables to show at which position in worldwide GDP (General for Public Expenditures and Per Capita for Private ones) each of these were in 2019.

```{r}
gdp_rank <- df %>%
  filter(Series.Code == "NY.GDP.MKTP.CD" & Year == 2019) %>%
  arrange(desc(Results))

gdp_rank$GDP.Rank <- rank(-gdp_rank$Results, ties.method = "first")

gdp_pc_rank <- df %>%
  filter(Series.Code == "NY.GDP.PCAP.CD" & Year == 2019) %>%
  arrange(desc(Results))

gdp_pc_rank$GDP.Rank <- rank(-gdp_pc_rank$Results, ties.method = "first")

top_expend_public <- df %>%
  filter(Series.Code == "SH.XPD.GHED.PC.CD" & Year == 2019) %>%
  arrange(desc(Results))

top_10_public <- merge(x = top_expend_public, y = gdp_rank[c(2, 7)], by = "Country.Code", all.x = TRUE) %>% 
  arrange(desc(Results)) %>% 
  slice_head(n = 10)

top_expend_private <- df %>%
  filter(Series.Code == "SH.XPD.PVTD.PC.CD" & Year == 2019) %>%
  arrange(desc(Results))

top_10_private <- merge(x = top_expend_private, y = gdp_pc_rank[c(2, 7)], by = "Country.Code", all.x = TRUE) %>% 
  arrange(desc(Results)) %>% 
  slice_head(n = 10)

bottom_expend_public <- df %>%
  filter(Series.Code == "SH.XPD.GHED.PC.CD" & Year == 2019) %>%
  arrange(desc(Results))

bottom_10_public <- merge(x = bottom_expend_public, y = gdp_rank[c(2, 7)], by = "Country.Code", all.x = TRUE) %>% 
  arrange(desc(Results)) %>% 
  slice_tail(n = 10)

bottom_expend_private <- df %>%
  filter(Series.Code == "SH.XPD.PVTD.PC.CD" & Year == 2019) %>%
  arrange(desc(Results))

bottom_10_private <- merge(x = bottom_expend_private, y = gdp_pc_rank[c(2, 7)], by = "Country.Code", all.x = TRUE) %>% 
  arrange(desc(Results)) %>% 
  slice_tail(n = 10)
```

```{r}
ggplot(data = top_10_public, aes(x = reorder(Country.Code, -Results), y = Results)) +
  geom_bar(stat="identity") +
  labs(title = "Top 10 Public Expenditures", subtitle = "In US$ per capita. GDP Rank over bars.", x = "Country", y = "Expenditure") +
  geom_text(aes(label = GDP.Rank), position = position_dodge(width = 0.5), vjust = -0.25)

ggplot(data = top_10_private, aes(x = reorder(Country.Code, -Results), y = Results)) +
  geom_bar(stat="identity") +
  labs(title = "Top 10 Private Expenditures", subtitle = "In US$ per capita. GDP Rank per capita over bars.", x = "Country", y = "Expenditure") +
  geom_text(aes(label = GDP.Rank), position = position_dodge(width = 0.5), vjust = -0.25)
```

The results above show that among the top 10 countries in public spending, only 2 of them belonged to the top 10 biggest GDPs in the world. The same pattern occurred for private expenditures pocketed by citizens, indicating as expected that the richest countries/citizens usually won't be the biggest spenders in health care systems, since they possess in general better life conditions that generate less sources for health issues.

Regarding the bottom 10 countries/citizens with the least expenditure in health care, in both systems the results were expected since most countries are among the poorest. 

To further the economical analysis of health care expenditure, it was generated a new variable to measure the percentual variation in health care expenditure per capita between 2001 and 2019, both in public and private systems, to verify which countries and citizens increased the most this ratio during the period. Only top 50 records in both scenarios were considered at the analysis, since the bigger the economy becomes the greater the potential expenditure in health can be.

```{r}
public_var <- df %>%
  filter(Series.Code == "SH.XPD.GHED.PC.CD", Year %in% c("2001", "2019")) %>%
  group_by(Country.Code) %>%
  filter(n() == 2) %>%
  mutate(expense_var_percent = ifelse(Year == "2019", round((Results / lag(Results) - 1) * 100, 2), 0)) %>%
  filter(expense_var_percent != 0) %>%
  select(Country.Name, Country.Code, expense_var_percent) %>%
  ungroup() %>%
  arrange(desc(expense_var_percent))

private_var <- df %>%
  filter(Series.Code == "SH.XPD.PVTD.PC.CD", Year %in% c("2001", "2019")) %>%
  group_by(Country.Code) %>%
  filter(n() == 2) %>%
  mutate(expense_var_percent = ifelse(Year == "2019", round((Results / lag(Results) - 1) * 100, 2), 0)) %>%
  filter(expense_var_percent != 0) %>%
  select(Country.Name, Country.Code, expense_var_percent) %>%
  ungroup() %>%
  arrange(desc(expense_var_percent))
```

Additionally, it was calculated the percentual variation in the GDP (general and per capita respectively) between the same years (2001-2019) to further compare with the increase in expenditure variations calculated above, merging the data frames.

```{r}
gdp_var <- df %>%
  filter(Series.Code == "NY.GDP.MKTP.CD", Year %in% c("2001", "2019")) %>%
  group_by(Country.Code) %>%
  filter(n() == 2) %>%
  mutate(gdp_var_percent = ifelse(Year == "2019", round((Results / lag(Results) - 1) * 100, 2), 0)) %>%
  filter(gdp_var_percent != 0) %>%
  select(Country.Name, Country.Code, gdp_var_percent) %>%
  ungroup() %>%
  arrange(desc(gdp_var_percent))

gdp_pc_var <- df %>%
  filter(Series.Code == "NY.GDP.PCAP.CD", Year %in% c("2001", "2019")) %>%
  group_by(Country.Code) %>%
  filter(n() == 2) %>%
  mutate(gdp_pc_var_percent = ifelse(Year == "2019", round((Results / lag(Results) - 1) * 100, 2), 0)) %>%
  filter(gdp_pc_var_percent != 0) %>%
  select(Country.Name, Country.Code, gdp_pc_var_percent) %>%
  ungroup() %>%
  arrange(desc(gdp_pc_var_percent))

gdp_pub_expense_merge <- merge(public_var, gdp_var, all.public_var = TRUE) %>% arrange(desc(expense_var_percent))

gdp_prv_expense_merge <- merge(private_var, gdp_pc_var, all.private_var = TRUE) %>% arrange(desc(expense_var_percent))

head(gdp_pub_expense_merge, n = 30)
head(gdp_prv_expense_merge, n = 30)
```

From the results above, in terms of public health care expenditure variation from 2001 to 2019, almost all countries in the top 50 were consistent in strongly increasing their spending beyond their economic growth at the period. A different pattern in private health expenditures happened, since the differences were timider and this superiority pattern only followed at the top 26 observations, becoming afterwards relevantly inconsistent, demonstrating that while countries in an economical growth pattern can afford greater investments in health care, individual citizens with a growing wealth address health care demands with a more concise approach.

Now, the second research aspect of infant and adult mortality comes into focus, with the ranking of which countries possess the best results in these indicators (equivalent to the smallest values) at the most recent year available (2019). Regarding the adult's rank, it required a calculation to measure the arithmetic average mortality ratio between males and females, since those indicators were separated originally, which was acceptable due to them having the same scale (per 1000 citizens).

```{r}
mort_inf_rank <- df %>%
  filter(Series.Code == "SP.DYN.IMRT.IN") %>%
  filter(Year == "2019") %>%
  rename(avg_inf_mort = Results) %>%
  select(Country.Name, Country.Code, avg_inf_mort) %>%
  arrange(desc(avg_inf_mort)) %>%
  mutate(mort.rank = row_number())

mort_adult_rank <- df %>%
  filter(Series.Code == "SP.DYN.AMRT.FE" | Series.Code == "SP.DYN.AMRT.MA") %>%
  filter(Year == "2019") %>%
  mutate(avg_adult_mort = ifelse(Series.Code == "SP.DYN.AMRT.FE", round((Results + lead(Results)) / 2, 2), 0)) %>%
  filter(avg_adult_mort != 0) %>%
  select(Country.Name, Country.Code, Year, avg_adult_mort) %>%
  arrange(desc(avg_adult_mort)) %>%
  mutate(mort.rank = row_number())

mort_adult_rank2 <- df %>%
  filter(Series.Code == "SP.DYN.AMRT.FE" | Series.Code == "SP.DYN.AMRT.MA") %>%
  filter(Year == "2001") %>%
  mutate(avg_adult_mort = ifelse(Series.Code == "SP.DYN.AMRT.FE", round((Results + lead(Results)) / 2, 2), 0)) %>%
  filter(avg_adult_mort != 0) %>%
  select(Country.Name, Country.Code, Year, avg_adult_mort) %>%
  arrange(desc(avg_adult_mort)) %>%
  mutate(mort.rank = row_number())
```

In the sequence, the 2001-2019 mortality ratio percentual variation was calculated to analyze which countries had an improvement or worsening at the period. Complementarily, the public and private variations in expenditures on health care calculated before were aggregated to the analysis, to check whether the countries with the greatest expenditure were the ones with the greatest decrease at mortality rates.

```{r}
mort_inf_var <- df %>%
  filter(Series.Code == "SP.DYN.IMRT.IN") %>%
  filter(Year == "2001" | Year == "2019") %>%
  mutate(mort_inf_var_percent = ifelse(Year == "2019", round((Results / lag(Results) - 1) * 100, 2), 0)) %>%
  filter(mort_inf_var_percent != 0) %>%
  select(Country.Name, Country.Code, mort_inf_var_percent) %>%
  arrange(desc(mort_inf_var_percent))

mort_adult_var <- rbind(mort_adult_rank, mort_adult_rank2) %>%
  arrange(Country.Name, Year) %>%
  group_by(Country.Code) %>%
  filter(n() == 2)  

mort_adult_var <- mort_adult_var %>%
  mutate(mort_adult_var_percent = ifelse(Year == "2019", round((avg_adult_mort / lag(avg_adult_mort) - 1) * 100, 2), 0)) %>%
  filter(mort_adult_var_percent != 0) %>%
  select(Country.Name, Country.Code, mort_adult_var_percent) %>%
  arrange(desc(mort_adult_var_percent))

infant_rank_var_merge <- merge(x = mort_inf_rank,  y = mort_inf_var[c(2, 3)], by = "Country.Code", all.x = TRUE)
infant_rank_var_merge <- merge(x = infant_rank_var_merge,  y = public_var[c(2, 3)], by = "Country.Code", all.x = TRUE)
infant_rank_var_merge <- merge(x = infant_rank_var_merge,  y = private_var[c(2, 3)], by = "Country.Code", all.x = TRUE) %>% 
  arrange(desc(avg_inf_mort)) %>%
  rename(public_expend_var = expense_var_percent.x, private_expend_var = expense_var_percent.y)

adult_rank_var_merge <- merge(x = mort_adult_rank[c(1, 2, 4, 5)], y = mort_adult_var[c(2, 3)], by = "Country.Code", all.x = TRUE)

adult_rank_var_merge <- merge(x = adult_rank_var_merge, y = public_var[c(2, 3)], by = "Country.Code", all.x = TRUE)

adult_rank_var_merge <- merge(x = adult_rank_var_merge, y = private_var[c(2, 3)], by = "Country.Code", all.x = TRUE) %>% 
  arrange(desc(avg_adult_mort)) %>%
  rename(public_expend_var = expense_var_percent.x, private_expend_var = expense_var_percent.y)

head(infant_rank_var_merge, n = 30)
head(adult_rank_var_merge, n = 30)
```

Analyzing the results, both in terms of infantile and adult mortality, almost all countries analyzed demonstrated a decrease at the period, ranging from 2% to 82%. Only 9 countries faced an increase at infant mortality and 5 at adult mortality. In general, all countries had a relevant increase at both public and private expenditures in health care during the period, indicating that as the most likely cause of such improvements, but a few outliers achieved that in a divergent scenario.

At the infant case, Cameroon and Guinea-Bissau had a decrease at government health expenditures per capita and an increase at private expenditures but managed to decrease their mortality ratio. Other countries had a reduction in out-of-pocket private health care per capita, but they counterbalanced it with an increase at public expending.

Looking at the other spectrum, with adult's mortality, the same pattern occurred, but a different and unique outlier appeared, at the case of Venezuela that suffered a reduction in both public and private expenditures but somehow diminished their adult mortality rate. Since this is a very specific case, a deeper investigation should be applied at the data source to verify and confirm such numbers and then decide on later procedures to analyze the root cause of this phenomenon.

Finally, the third aspect of analysis at this study was about life expectancy (LE). To start with, a rank of countries from highest to lowest LE was generated, considering data from the most recent available year - 2019. 

```{r}
life_expec_rank <- df %>%
  filter(Series.Code == "SP.DYN.LE00.IN") %>%
  filter(Year == "2019") %>%
  rename(avg_life_expec = Results) %>%
  select(Country.Name, Country.Code, avg_life_expec) %>%
  arrange(desc(avg_life_expec)) %>%
  mutate(le.rank = row_number())
```

Next, the percentual variation of life expectancy between 2001 and 2019 was calculated to evaluate how countries evolved over time on the matter. These variables were then merged at the data frame with the public and private expenditures variations at the same period.

```{r}
life_expec_var <- df %>%
  filter(Series.Code == "SP.DYN.LE00.IN") %>%
  filter(Year == "2001" | Year == "2019") %>%
  group_by(Country.Code) %>%
  filter(n() == 2) %>%
  mutate(le_var_percent = ifelse(Year == "2019", round((Results / lag(Results) - 1) * 100, 2), 0)) %>%
  filter(le_var_percent != 0) %>%
  select(Country.Name, Country.Code, le_var_percent) %>%
  ungroup() %>%
  arrange(desc(le_var_percent))

life_expec_merge <- merge(x = life_expec_rank, y = life_expec_var[c(2, 3)], by = "Country.Code", all.x = TRUE)

life_expec_merge <- merge(x = life_expec_merge, y = public_var[c(2, 3)], by = "Country.Code", all.x = TRUE)

life_expec_merge <- merge(x = life_expec_merge,  y = private_var[c(2, 3)], by = "Country.Code", all.x = TRUE) %>% 
  arrange(desc(le_var_percent)) %>%
  rename(public_expend_var = expense_var_percent.x, private_expend_var = expense_var_percent.y)

head(life_expec_merge, n = 30)
```

Data showed that almost all countries increased their life expectancy rates at the period, apart from three (Syria, Grenada and Venezuela). Similarly to the mortality rates analyzed before, countries that improved had an increase in either or both public and private health expenditures, but the outlier in this case was Grenada, that had both expenditures increased during the period but wasn't able to improve their average life expectancy, which demands further analysis of lurking variables that weren't present or considered in this study to explain this case.

At last, to better visualize the relationship between health care expenditure and life expectancy, both variables were plotted to scatter plots, where a trend was observed when expenditures hit at least US$ 2000, leading to a life expectancy of 80 plus years.

```{r}
pub_expend_le_corr <- merge(x = top_expend_public, y = life_expec_rank, all.x = TRUE) %>% drop_na(le.rank)

prv_expend_le_corr <- merge(x = top_expend_private, y = life_expec_rank, all.x = TRUE) %>% drop_na(le.rank)

ggplot(data = pub_expend_le_corr, aes(x = Results, y = avg_life_expec)) +
  geom_point(mapping = aes(colour = avg_life_expec), alpha = 0.5) +
  labs(title = "Public Expenditure by Life Expectancy Dispersion", subtitle = "Expenditure in US$ per capita.", x = "Expenditure", y = "LE")

ggplot(data = prv_expend_le_corr, aes(x = Results, y = avg_life_expec)) +
  geom_point(mapping = aes(colour = avg_life_expec), alpha = 0.5) +
  labs(title = "Private Expenditure by Life Expectancy Dispersion", subtitle = "Expenditure in US$ per capita.", x = "Expenditure", y = "LE")
```

## Conclusions

After analyzing the three aspects delimited at the beginning of this document, some conclusions were drawn, starting with the fact that the richest countries are not necessarily the ones spending the most on health care per capita, being it at the public or private system. In fact, among the top 10 public spenders, 8 countries had a population below 30 million citizens, which facilitates such an extensive expenditure per capita, but highlights the other 2 countries (United States of America and Germany) that had a significant spending related to their population size. For top private expenditures it followed as expected, embracing rich countries at the 'podium'.

Regarding the mortality indicators, data exemplifies that a potential requirement for improvements in mortality rates is the increase of health care expenditures per capita, being it either on the public or private system, since countries that consistently upgraded their expenditures in health care achieved lower levels of infant and adult mortality.

Finally, addressing the life expectancy indicator, it followed the same pattern as with mortality rates, in which countries evolved over time by either increasing the public budgets or providing more wealth to their citizens to enable them to hire better health care services that can expand their life spans.

For future studies on the topic, it would be optimal to aggregate more recent data, especially through and after the Covid-19 pandemic, to evaluate whether countries that spent more in their health care systems were able to translate those investments in a better treatment and cure ratio, while considering the concurrently the regular social indications of health and social welfare.
