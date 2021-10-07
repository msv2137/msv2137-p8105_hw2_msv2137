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
