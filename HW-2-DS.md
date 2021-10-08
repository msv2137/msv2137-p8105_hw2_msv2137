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
  read.csv(pols_month_path) %>%
  janitor::clean_names() %>%
  separate(mon, into = c("year","month","day")) %>%
  mutate(month = month.name[as.integer(month)],
         president = party[prez_gop + 1]) %>%
  select(year, month, president, everything(), -day, -prez_dem, -prez_gop)

#view completed dataset
#View(pols_data)
```
