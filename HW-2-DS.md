HW 2
================
Malvika Venkataraman

# Problem 1

## Read in the Trash Wheel Dataset and Clean

``` r
trash_wheel_path = "./data/Trash-Wheel-Collection-Totals-7-2020-2.xlsx"

trash_wheel_df = read_excel(trash_wheel_path, sheet = "Mr. Trash Wheel", range = "A2:N534") %>%
  janitor::clean_names() %>%
  filter(dumpster != "NA") %>%
  mutate(sports_balls = as.integer(sports_balls))

#view dataset
#View(trash_wheel_df)
```

## Read in the Precipitation Data (2018 and 2019) and Clean

### Read and clean 2018 precipitation data

``` r
precip_data_2018 = read_excel(trash_wheel_path, sheet = "2018 Precipitation", range = "A2:B14") %>%
  janitor::clean_names() %>%
  filter(total != "NA") %>%
  mutate(year = 2018)

#View(precip_data_2018)
```

### Read and clean 2019 precipitation data

``` r
precip_data_2019 = read_excel(trash_wheel_path, sheet = "2019 Precipitation", range = "A2:B14") %>%
  janitor::clean_names() %>%
  filter(total != "NA") %>%
  mutate(year = 2019)

#View(precip_data_2019)
```

### Combine 2018 and 2019 datasets

``` r
precip_data = bind_rows(precip_data_2018, precip_data_2019) %>%
  mutate(month = month.name[month]) %>%
  select(year, month, total)

#View(precip_data)
```

## Interpretation

In the Mr. Trash Wheel dataset, there are 453 rows and 14 columns. There
are 453 observations between 5/16/14 and 1/4/21, for the seven
categories of trash represented in the dataset. Key variables include
the Weight of the trash (in tons), and the Volume (in cubic yards). The
median number of sports balls in a dumpster in 2019 was 9.

In the 2018 and 2019 Merged Precipitation dataset, there are 24 rows and
3 columns. There are 24 observations between 1/2018 and 12/2019. Key
variables include the month, year and the total precipitation, for each
month. For the available data, the total precipitation in 2018 was 70.33

# Problem 2

## Read in the FiveThirtyEight Datasets and Clean

### File Paths

``` r
pols_month_path = "./data/fivethirtyeight_datasets/pols-month.csv"
unemployment_path = "./data/fivethirtyeight_datasets/unemployment.csv"
snp_path = "./data/fivethirtyeight_datasets/snp.csv"
```

### Pols Month Dataset

``` r
#read and clean pols-month.csv
party = c("dem","gop")

pols_data =
  read_csv(pols_month_path) %>%
  janitor::clean_names() %>%
  separate(mon, into = c("year","month","day")) %>%
  mutate(month = month.name[as.integer(month)],
         president = party[prez_gop + 1]) %>%
  select(year, month, president, everything(), -day, -prez_dem, -prez_gop)
```

    ## Rows: 822 Columns: 9

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl  (8): prez_gop, gov_gop, sen_gop, rep_gop, prez_dem, gov_dem, sen_dem, r...
    ## date (1): mon

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#view completed dataset
#View(pols_data)
```

### SNP Dataset

``` r
#read and convert dates to full year to match the pols month and unemployment csv's
snp_data <- read_csv(snp_path)
```

    ## Rows: 787 Columns: 2

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (1): date
    ## dbl (1): close

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
dates <- snp_data$date
dates <- as.Date(dates, "%m/%d/%y")
dates <- as.Date(ifelse(dates > Sys.Date(), format(dates, "19%y-%m-%d"), format(dates)))
snp_data[,"date"] <- dates 

#clean snp.csv
snp_data =
  snp_data %>%
  janitor::clean_names() %>%
  separate(date, into = c("year","month","day")) %>%
  mutate(month = month.name[as.integer(month)]) %>%
  select(year, month, snp_index = close)

#view completed dataset
#View(snp_data)
```

### Unemployment Dataset

``` r
#read and clean unemployment csv
unemployment_data =
  read_csv(unemployment_path) %>%
  pivot_longer(
    Jan:Dec,
    names_to = "month",
    values_to = "percentage") %>%
  mutate(month = month.name[match(pull(.,month),month.abb)],
         Year = as.character(Year)) %>%
  select(year =  Year, month, unemployment_percentage = percentage) #change name to make it more clear
```

    ## Rows: 68 Columns: 13

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## dbl (13): Year, Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#view completed dataset
#View(unemployment_data)
```

### Merge the Datasets

``` r
#merge
fivethirtyeight_merged_data =
  left_join(pols_data, snp_data, by = c("year","month")) %>%
  left_join(unemployment_data, by = c("year","month"))

#view completed dataset
#View(fivethirtyeight_merged_data)
```

### Results

The `pols_data` has 822 rows and 9 columns. It indicates, for a specific
year and month, whether the president was a republican (`gop`) or a
democrat (`dem`). It also indicates the number of republican or
democratic governors (`gov_gop`, `gov_dem`), as well as representatives
(`rep_gop`, `rep_dem`). The range of years the dataset includes is 1947
to 2015.

The `snp_data` has 787 rows and 3 columns. It indicates the closing
values of the S&P stock index at the beginning of a designated year and
month. Key variables include the year, month and S&P index. The range of
years the dataset includes is 2015 to 1950.

The `unemployment_data` has 816 rows and 3 columns. It indicates, the
unemployment percentage at the beginning of a designated year and month.
Key variables include the year, month and unemployment percentage. The
range of years the dataset includes is 1948 to 2015.

The resulting dataset (`fivethirtyeight_merged_data`) has 822 rows and
11 columns. The years in the dataset range from 1947 to 2015. Key
variables include the year and the month. The resulting dataset provides
information on the political party of the president, the number of
republican or democratic governors, senators and representatives, the
closing values of the S&P stock index, and the unemployment percentage
for a specific year and month.

# Problem 3

## Load baby name dataset

``` r
#load, read, view data
baby_name_path = "./data/Popular_Baby_Names.csv"

baby_name_df = read_csv(baby_name_path)
```

    ## Rows: 19418 Columns: 6

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (3): Gender, Ethnicity, Child's First Name
    ## dbl (3): Year of Birth, Count, Rank

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#View(baby_name_df)
```

## Check for issues

``` r
unique(baby_name_df[c("Year of Birth")]) #no issues
```

    ## # A tibble: 6 × 1
    ##   `Year of Birth`
    ##             <dbl>
    ## 1            2016
    ## 2            2015
    ## 3            2014
    ## 4            2013
    ## 5            2012
    ## 6            2011

``` r
unique(baby_name_df[c("Gender")]) #no issues
```

    ## # A tibble: 2 × 1
    ##   Gender
    ##   <chr> 
    ## 1 FEMALE
    ## 2 MALE

``` r
unique(baby_name_df[c("Ethnicity")]) #issues
```

    ## # A tibble: 7 × 1
    ##   Ethnicity                 
    ##   <chr>                     
    ## 1 ASIAN AND PACIFIC ISLANDER
    ## 2 BLACK NON HISPANIC        
    ## 3 HISPANIC                  
    ## 4 WHITE NON HISPANIC        
    ## 5 ASIAN AND PACI            
    ## 6 BLACK NON HISP            
    ## 7 WHITE NON HISP

## Tidy the data

``` r
baby_name_df =
  read_csv(baby_name_path) %>%
  janitor::clean_names() %>%
  distinct() %>%
  mutate(
    gender = str_to_lower(gender),
    ethnicity = str_to_lower(ethnicity),
    childs_first_name = str_to_lower(childs_first_name),
    ethnicity = replace(ethnicity, ethnicity == "asian and paci",
                        "asian and pacific islander"),
    ethnicity = replace(ethnicity, ethnicity == "black non hisp",
                        "black non hispanic"),
    ethnicity = replace(ethnicity, ethnicity == "white non hisp",
                        "white non hispanic")
    )
```

    ## Rows: 19418 Columns: 6

    ## ── Column specification ────────────────────────────────────────────────────────
    ## Delimiter: ","
    ## chr (3): Gender, Ethnicity, Child's First Name
    ## dbl (3): Year of Birth, Count, Rank

    ## 
    ## ℹ Use `spec()` to retrieve the full column specification for this data.
    ## ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.

``` r
#view completed dataset
#View(baby_name_df)
```

## Table: Popularity of Name Olivia

``` r
olivia_table_df =
  filter(baby_name_df, gender == "female", childs_first_name == "olivia") %>%
  select(year_of_birth, ethnicity, rank) %>%
  pivot_wider(
    names_from = "year_of_birth",
    values_from = "rank"
  )

knitr::kable(olivia_table_df, format = "html", 
             caption = 
               "Table 1: Popularity Rank of 'Olivia' as a Female Baby Name Over Time")
```

<table>
<caption>
Table 1: Popularity Rank of ‘Olivia’ as a Female Baby Name Over Time
</caption>
<thead>
<tr>
<th style="text-align:left;">
ethnicity
</th>
<th style="text-align:right;">
2016
</th>
<th style="text-align:right;">
2015
</th>
<th style="text-align:right;">
2014
</th>
<th style="text-align:right;">
2013
</th>
<th style="text-align:right;">
2012
</th>
<th style="text-align:right;">
2011
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
asian and pacific islander
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
3
</td>
<td style="text-align:right;">
4
</td>
</tr>
<tr>
<td style="text-align:left;">
black non hispanic
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
6
</td>
<td style="text-align:right;">
8
</td>
<td style="text-align:right;">
10
</td>
</tr>
<tr>
<td style="text-align:left;">
hispanic
</td>
<td style="text-align:right;">
13
</td>
<td style="text-align:right;">
16
</td>
<td style="text-align:right;">
16
</td>
<td style="text-align:right;">
22
</td>
<td style="text-align:right;">
22
</td>
<td style="text-align:right;">
18
</td>
</tr>
<tr>
<td style="text-align:left;">
white non hispanic
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
1
</td>
<td style="text-align:right;">
4
</td>
<td style="text-align:right;">
2
</td>
</tr>
</tbody>
</table>

## Table: Most Popular Name Among Male Children

``` r
pop_male_name_df =
  filter(baby_name_df, gender == "male", rank == "1") %>%
  select(year_of_birth, ethnicity, childs_first_name) %>%
  pivot_wider(
    names_from = "year_of_birth",
    values_from = "childs_first_name"
  )

knitr::kable(pop_male_name_df, format = "html", 
             caption = 
               "Table 2: All Time Most Popular Name Among Male Children")
```

<table>
<caption>
Table 2: All Time Most Popular Name Among Male Children
</caption>
<thead>
<tr>
<th style="text-align:left;">
ethnicity
</th>
<th style="text-align:left;">
2016
</th>
<th style="text-align:left;">
2015
</th>
<th style="text-align:left;">
2014
</th>
<th style="text-align:left;">
2013
</th>
<th style="text-align:left;">
2012
</th>
<th style="text-align:left;">
2011
</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align:left;">
asian and pacific islander
</td>
<td style="text-align:left;">
ethan
</td>
<td style="text-align:left;">
jayden
</td>
<td style="text-align:left;">
jayden
</td>
<td style="text-align:left;">
jayden
</td>
<td style="text-align:left;">
ryan
</td>
<td style="text-align:left;">
ethan
</td>
</tr>
<tr>
<td style="text-align:left;">
black non hispanic
</td>
<td style="text-align:left;">
noah
</td>
<td style="text-align:left;">
noah
</td>
<td style="text-align:left;">
ethan
</td>
<td style="text-align:left;">
ethan
</td>
<td style="text-align:left;">
jayden
</td>
<td style="text-align:left;">
jayden
</td>
</tr>
<tr>
<td style="text-align:left;">
hispanic
</td>
<td style="text-align:left;">
liam
</td>
<td style="text-align:left;">
liam
</td>
<td style="text-align:left;">
liam
</td>
<td style="text-align:left;">
jayden
</td>
<td style="text-align:left;">
jayden
</td>
<td style="text-align:left;">
jayden
</td>
</tr>
<tr>
<td style="text-align:left;">
white non hispanic
</td>
<td style="text-align:left;">
joseph
</td>
<td style="text-align:left;">
david
</td>
<td style="text-align:left;">
joseph
</td>
<td style="text-align:left;">
david
</td>
<td style="text-align:left;">
joseph
</td>
<td style="text-align:left;">
michael
</td>
</tr>
</tbody>
</table>

## Scatterplot

``` r
name_popularity_plot_df = 
  filter(
    baby_name_df,
    year_of_birth == "2016",
    gender == "male",
    ethnicity == "white non hispanic") %>%
  select(rank, count)

#plot
ggplot(name_popularity_plot_df, aes(x = rank, y = count)) +
  geom_point() +
  labs(
    title = 
      "Plot of Name Popularity for White, Non-Hispanic, Male Children Born in 2016",
    x = "Rank in Popularity of a Name",
    y = "Number of Children With a Name"
  )
```

![](HW-2-DS_files/figure-gfm/unnamed-chunk-15-1.png)<!-- -->
