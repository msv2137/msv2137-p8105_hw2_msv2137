HW 2
================
Malvika Venkataraman

# Problem 1

## Read in the Trash Wheel Dataset and Clean

``` r
trash_wheel_path = "./data/Trash-Wheel-Collection-Totals-8-6-19.xlsx"

trash_wheel_df = read_excel(trash_wheel_path, sheet = "Mr. Trash Wheel", range = "A2:N408") %>%
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

*insert here*

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
View(snp_data)
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

*insert text here*

# Problem 3

## Load and tidy baby name dataset

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
View(baby_name_df)
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
