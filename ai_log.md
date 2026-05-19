# ai_log.md

## AI Tool Used
ChatGPT (OpenAI)

---

## Prompt 1: Understanding the Lab Task

**Prompt:**

I have a Quarto `.qmd` lab about tidying ASEAN meat production data from an Excel file. Can you explain what this lab is asking me to do in simple terms?

**Useful AI Output:**

AI explained that the lab requires us to:
- import meat production data from Excel,
- identify why the data is not tidy,
- clean it into tidy format,
- clean population data,
- join both datasets,
- calculate meat production per capita,
- and create a visualization.

---

## Prompt 2: Why Sheet X.14 Is Not Tidy

**Prompt:**

Why is this ASEAN meat production Excel sheet not tidy if years and countries are used as headers or labels?

**Useful AI Output:**

AI explained that the sheet is not tidy because:
- years are stored as labels instead of a proper column,
- countries are spread across multiple columns,
- animal types are mixed into row labels,
- ASEAN total/summary data is mixed with country-level data,
- missing values are shown using symbols like `-`.

This helped us write the explanation for Section 3.1 Part C.

---

## Prompt 3: Importing the Excel Sheet

**Prompt:**

How do I read only a specific range from an Excel sheet in R using `readxl`?

**Code Suggested / Used:**

```r
meat_raw <- read_xlsx(
  path = "ASEAN-Statistical-Yearbook-2023.xlsx",
  sheet = "X.14",
  range = "A4:L36"
)

meat_raw
Reflection:

This was useful because the Excel file contained extra title rows, notes, and formatting that should not be imported as data.

Prompt 4: Renaming the First Column

Prompt:

In R tidyverse, how do I rename the first column of a dataframe when the column name is messy or unknown?

Code Suggested / Used:

meat_raw %>%
  rename(label = 1)

Reflection:

This helped because the first column contained mixed labels such as years and animal names.

Prompt 5: Removing Empty Columns

Prompt:

How do I remove columns that are completely empty in a tidyverse pipeline?

Code Suggested / Used:

select(where(~ !all(is.na(.x))))

Reflection:

This was useful because the Excel import may contain blank columns caused by spreadsheet formatting.

Prompt 6: Removing ASEAN Total Column

Prompt:

How do I remove the ASEAN total column from my dataset if I only want individual countries?

Code Suggested / Used:

select(-any_of("ASEAN"))

Reflection:

This helped remove aggregate data so that the final dataset only contained country-level observations.

Prompt 7: Extracting Year Values

Prompt:

How can I detect rows where the label is a year like 2018, 2019, 2020, or 2021, then create a year column in R?

Code Suggested / Used:

mutate(
  label = str_squish(as.character(label)),
  year = as.numeric(if_else(
    str_detect(label, "^\\d{4}$"),
    label,
    NA_character_
  ))
)

Reflection:

This helped convert year labels from the Excel layout into a proper variable column.

Prompt 8: Filling Year Downward

Prompt:

In a messy Excel table, the year appears once above several rows. How do I fill the year down to the rows below it using tidyverse?

Code Suggested / Used:

fill(year, .direction = "down")

Reflection:

This was useful because each animal row needed to be linked to the correct year.

Prompt 9: Creating Animal Categories

Prompt:

How do I create an animal column based on text labels like beef, pig, mutton, goat, and poultry?

Code Suggested / Used:

mutate(
  label_lower = str_to_lower(label),
  animal = case_when(
    str_detect(label_lower, "beef") &
      str_detect(label_lower, "buffalo") ~ "beef and buffalo",
    str_detect(label_lower, "^pig") ~ "pig",
    str_detect(label_lower, "mutton") |
      str_detect(label_lower, "goat") ~ "mutton and goat",
    str_detect(label_lower, "^poultry") &
      !str_detect(label_lower, "production") ~ "poultry",
    TRUE ~ NA_character_
  )
)

Reflection:

This helped standardise animal type names into consistent categories.

Prompt 10: Filtering Only Useful Rows

Prompt:

How do I keep only rows where animal type exists and the year is from 2018 to 2021?

Code Suggested / Used:

filter(!is.na(animal), year %in% 2018:2021)

Reflection:

This removed irrelevant title, blank, and source rows.

Prompt 11: Converting Wide Country Columns into Long Format

Prompt:

My Excel data has countries as columns. How do I convert it into tidy long format with one country column and one production column?

Code Suggested / Used:

pivot_longer(
  cols = -c(label, year, animal),
  names_to = "country",
  values_to = "production",
  values_transform = list(production = as.character)
)

Reflection:

This was one of the most useful AI suggestions because it changed the data into tidy format.

Prompt 12: Handling Missing Values Shown as Dash

Prompt:

Some values in my Excel file are shown as -. How do I convert them into NA while converting production values to numeric?

Code Suggested / Used:

mutate(
  country = str_squish(country),
  production = readr::parse_number(production, na = c("-", ""))
)

Reflection:

This fixed the problem where missing values were represented as symbols instead of real NA values.

Prompt 13: Arranging and Selecting Final Columns

Prompt:

How do I arrange my tidy meat dataset by year, animal, and country, then keep only the final columns?

Code Suggested / Used:

arrange(desc(year), animal, country) %>%
select(country, animal, year, production)

Reflection:

This made the final output cleaner and easier to compare with the reference dataset.

Prompt 14: Full Meat Tidying Code

Prompt:

Can you help me combine all the cleaning steps into one tidyverse pipeline for the ASEAN meat production data?

Code Used:

meat_tidy <- meat_raw %>%
  rename(label = 1) %>%
  select(where(~ !all(is.na(.x)))) %>%
  select(-any_of("ASEAN")) %>%
  mutate(
    label = str_squish(as.character(label)),
    year = as.numeric(if_else(
      str_detect(label, "^\\d{4}$"),
      label,
      NA_character_
    ))
  ) %>%
  fill(year, .direction = "down") %>%
  mutate(
    label_lower = str_to_lower(label),
    animal = case_when(
      str_detect(label_lower, "beef") &
        str_detect(label_lower, "buffalo") ~ "beef and buffalo",
      str_detect(label_lower, "^pig") ~ "pig",
      str_detect(label_lower, "mutton") |
        str_detect(label_lower, "goat") ~ "mutton and goat",
      str_detect(label_lower, "^poultry") &
        !str_detect(label_lower, "production") ~ "poultry",
      TRUE ~ NA_character_
    )
  ) %>%
  select(-label_lower) %>%
  filter(!is.na(animal), year %in% 2018:2021) %>%
  pivot_longer(
    cols = -c(label, year, animal),
    names_to = "country",
    values_to = "production",
    values_transform = list(production = as.character)
  ) %>%
  mutate(
    country = str_squish(country),
    production = readr::parse_number(production, na = c("-", ""))
  ) %>%
  arrange(desc(year), animal, country) %>%
  select(country, animal, year, production)
Prompt 15: Checking Row Count and Animal Types

Prompt:

How do I check whether my tidy meat dataset has the correct number of rows and animal categories?

Code Used:

nrow(meat_tidy)

meat_tidy %>%
  distinct(animal) %>%
  arrange(animal)

Reflection:

This helped verify that the dataset contained the expected 160 rows and four animal categories.

Prompt 16: Fixing Cambodia Mutton and Goat Values

Prompt:

In my dataset, Cambodia mutton and goat values from 2018 to 2021 are wrongly read as 0 but should be missing. How do I replace only those values with NA?

Code Used:

meat_tidy <- meat_tidy %>%
  mutate(
    production = if_else(
      country == "Cambodia" &
        animal == "mutton and goat" &
        year %in% 2018:2021 &
        production == 0,
      NA_real_,
      production
    )
  )

Reflection:

This was important because AI initially did not detect this dataset-specific issue automatically. The team corrected it after comparing with the source data.

Prompt 17: Comparing with Reference Data

Prompt:

How do I compare my cleaned dataset with a reference CSV file in R?

Code Used:

meat_ref <- read_csv("meat_tidy_reference.csv", show_col_types = FALSE)

compare(meat_tidy, meat_ref, tolerance = 1e-6)

Reflection:

This helped verify whether our cleaned data matched the expected answer.

Prompt 18: Importing Population Data

Prompt:

How do I import the population sheet from the ASEAN Excel file while skipping the title rows?

Code Used:

pop_raw <- read_xlsx(
  path = "ASEAN-Statistical-Yearbook-2023.xlsx",
  sheet = "I.1",
  skip = 4
)

pop_raw
Prompt 19: Tidying Population Data

Prompt:

How do I tidy a population table where countries are rows and years are columns?

Code Used:

pop_tidy <- pop_raw %>%
  rename(country = 1) %>%
  select(where(~ !all(is.na(.x)))) %>%
  mutate(country = str_squish(as.character(country))) %>%
  filter(!is.na(country), country != "", country != "ASEAN") %>%
  filter(
    !str_detect(country, "^Source"),
    !str_detect(country, "ASEAN Secretariat")
  ) %>%
  pivot_longer(
    cols = -country,
    names_to = "year",
    values_to = "pop"
  ) %>%
  mutate(
    year = as.numeric(year),
    pop = as.numeric(pop)
  ) %>%
  filter(year %in% 2013:2022) %>%
  arrange(desc(year), country) %>%
  select(country, year, pop)

Reflection:

This helped convert the population dataset into tidy format with one row per country-year.

Prompt 20: Comparing Population Data

Prompt:

How do I compare my population tidy dataset with the reference file?

Code Used:

pop_ref <- read_csv("population_tidy_reference.csv", show_col_types = FALSE)

compare(pop_tidy, pop_ref, tolerance = 1e-6)
Prompt 21: Joining Meat and Population Data

Prompt:

How do I join the meat production dataset with the population dataset by country and year?

Code Used:

merged <- meat_tidy %>%
  left_join(pop_tidy, by = c("country", "year")) %>%
  select(country, animal, year, production, pop)

Reflection:

This joined each meat production observation with the corresponding population value.

Prompt 22: Checking Row Order After Join

Prompt:

How do I check if the row order stayed the same after joining two datasets?

Code Used:

identical(
  merged %>% select(country, animal, year),
  meat_tidy %>% select(country, animal, year)
)

Reflection:

This helped ensure that the join did not accidentally reorder the meat production records.

Prompt 23: Calculating Per-Capita Meat Production

Prompt:

Production is in thousand metric tons and population is in thousand persons. How do I calculate kg per person?

AI Explanation:

Since both production and population are in thousands:

production / population = metric tons per person

To convert metric tons to kilograms:

metric tons per person × 1000 = kg per person

Code Used:

merged <- merged %>%
  mutate(meat_per_cap = production / pop * 1000)
Prompt 24: Comparing Merged Dataset

Prompt:

How do I compare my final merged dataset with the reference merged CSV?

Code Used:

merged_ref <- read_csv("merged_reference.csv", show_col_types = FALSE)

compare(merged, merged_ref, tolerance = 1e-6)
Prompt 25: Creating the Plot

Prompt:

How do I create a ggplot showing per-capita meat production over time by animal type for each ASEAN country?

Code Used:

ggplot(
  merged,
  aes(x = year, y = meat_per_cap, color = animal)
) +
  geom_line() +
  geom_point() +
  labs(
    x = NULL,
    y = "Production (kg per person)",
    color = "Animal",
    title = "Per-Capita Meat Production in ASEAN, 2018–2021",
    caption = "Source: ASEAN Statistical Yearbook 2023"
  ) +
  xlim(2017.9, 2021.1) +
  scale_y_log10(
    breaks = c(0.01, 0.1, 1, 10, 100),
    labels = c("0.01", "0.1", "1", "10", "100")
  ) +
  facet_wrap(~country) +
  theme(
    legend.position = "inside",
    legend.position.inside = c(1, 0),
    legend.justification.inside = c(1, 0.05)
  )

Reflection:

AI helped suggest using facet_wrap(~country) so each country could be compared separately.