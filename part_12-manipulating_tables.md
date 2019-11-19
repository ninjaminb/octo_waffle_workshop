Selecting and Manipulating Data
================

  - [Manipulating Rows (“Cases”)](#manipulating-rows-cases)
      - [Arranging Rows](#arranging-rows)
      - [Extracting Rows](#extracting-rows)
  - [Manipulating Columns
    (“Variables”)](#manipulating-columns-variables)
      - [Selecting Columns](#selecting-columns)
      - [Extracting and Arranging
        Columns](#extracting-and-arranging-columns)
      - [Making New Variables](#making-new-variables)
      - [New Variables from Non-Vectorized
        Functions](#new-variables-from-non-vectorized-functions)
  - [Hands-On Exercise](#hands-on-exercise)

In this section, you will learn how to select specific entries (rows)
based on their content from a table. Also, you will learn how to
retrieve specific variables (columns) from the table and/or add new
variables based on exisiting values to the data.

Having combined and tidied the plate data, we now have an object
`plate_data` containing

| sample\_id | replicate\_id | concentration | intensity |
| :--------: | :-----------: | :-----------: | :-------: |
|  control   |     rep.A     |       0       |  \-0.121  |
|  control   |     rep.A     |       1       |    NA     |
|  control   |     rep.A     |      10       |    NA     |
|  control   |     rep.A     |      100      |  \-0.340  |
|  control   |     rep.A     |     1000      |    NA     |
|  control   |     rep.B     |       0       |   0.232   |
|  control   |     rep.B     |       1       |    NA     |

*(table abridged)*

## Manipulating Rows (“Cases”)

We have already seen how to add cases using `add_row(...)`. Let’s see
how to arrange and extract rows based on their content.

### Arranging Rows

Tables can be sorted by specific columns. This is rather a convenience
for the user than a necessity to perform further manipulations.

``` r
plate_data %>% arrange(concentration, intensity)
plate_data %>% arrange(desc(concentration), intensity)
```

### Extracting Rows

These functions return a subset of rows as a new table.

``` r
# extract rows that meet criteria
plate_data %>% filter(sample_id == "treatment_B")
plate_data %>% filter(replicate_id == "rep.B" & concentration > 50)
plate_data %>% filter(intensity * concentration > 1e4)

plate_data %>% filter(intensity >= 1 & intensity <= 3)
plate_data %>% filter(between(intensity, 1, 3)) # same, shorter

plate_data %>% filter(near(intensity, 10, 0.5)) # safe for floating point numbers

# remove duplicate rows
plate_data %>% distinct(sample_id, concentration)
```

One can also select (randomly) a number of cases.

``` r
# randomly extract N rows without replacement
plate_data %>% sample_n(5, replace = FALSE)
# randomly extract % rows with replacement
plate_data %>% sample_frac(.1, replace = TRUE)

# select rows by position
plate_data %>% slice(4:6)

# order and select top N entries
plate_data %>% top_n(5, intensity)
```

## Manipulating Columns (“Variables”)

It might not be obvious at this point, but most power of the `tidyverse`
is taken from its ability to operate on existing columns and create new
columns.

### Selecting Columns

Many functions will require a proper selection of columns, e.g. during
data tidying. This is why we focus here on how to specify a column
selection before we move on to more illustrative examples.

> The `tidyverse` makes extensive use of ‘quasiquotation’. This means
> that you can refer e.g. to column names (which are usually of type
> `character`) without using quotation marks (`""` or `''`). But beware,
> some arguments must be `character` vectors. These must be quoted.

  - To select a column, you can always **type its name between quotation
    marks**. If you want to specify multiple columns, use a `character`
    vector with the column names.

  - Most functions will accept **unquoted column names**. You don’t need
    a vector or anything like that then.
    
    Typically, this is indicated with `...` in the ‘Usage’ section of
    the Documentation.

  - In addition, there are **helper functions** to select multiple
    columns based on their position or based on their name.
    
    They are documented in `?select_helpers`.
    
    | operator/function | selection                                        |
    | ----------------- | ------------------------------------------------ |
    | `everything`      | all columns                                      |
    | `last_col`        | last column                                      |
    | `one_of`          | these column names as character vector           |
    | `starts_with`     | column names starting with this prefix literally |
    | `ends_with`       | column names ending with this suffix literally   |
    | `contains`        | column names containing this string literally    |
    | `matches`         | column names matching the regular expression     |
    | `:`               | all columns between                              |
    | `-`               | all columns except                               |
    

    For ‘scoped’ summarizing (`summarize_at`) and mutating verbs
    (`mutate_at`), you will need to wrap the helper statement with
    `vars(...)`.
    
    For ‘scoped’ filtering verbs (`filter_all` or `filter_if`), there
    are additional variants called `all_vars(...)` and `any_vars(...)`.

### Extracting and Arranging Columns

Most frequently, you will need

  - `select` to (optionally rename and) extract columns as table, or
  - `pull` to extract column values of *one* column as a vector; which
    defaults to the last column is none is specified.

<!-- end list -->

``` r
# return (last) column as vector
plate_data %>% pull
# return selected columns as table
plate_data %>% select(intensity)   # with quasiquotation
plate_data %>% select("intensity") # with explicit names
plate_data %>% select(starts_with("int"))
plate_data %>% select(ends_with("id"))
plate_data %>% select(c("sample_id", "replicate_id"))
```

There are the ‘scoped’ variants,

  - `select_all` to rename all columns (e.g. make uppercase),
  - `select_at` to (optionally rename and) extract columns select with
    `vars(...)`, and
  - `select_if` to (optionally rename and) extract columns that meet a
    certain property, (e.g. contain numeric values).

<!-- end list -->

``` r
# select based e.g. on data type
plate_data %>% select_if(is.numeric)
# select columns and rename column names
plate_data %>% select_all(toupper)
plate_data %>% select_at(vars(ends_with("id")), ~str_c(., "iot"))
```

To rename columns only, there are `rename` variants of the above.

``` r
plate_data %>% rename("conc" = "concentration") # with explicit names
plate_data %>% rename(conc = concentration)     # with quasiquotation
plate_data %>% rename_at(vars(ends_with("id")), ~str_c(., c("ea", "iot")))
```

Less importantly, `select` can be used for re-arranging columns.

``` r
plate_data %>% select(concentration, ends_with("id"), intensity)
```

### Making New Variables

`mutate` and its ‘scoped’ variants will apply *vectorized* functions on
the selected columns and append a new column with the result of the
transformation.

`transmute` is a short-hand for `mutate` followed by `select` on the
result.

``` r
# create a new column ...
plate_data %>% mutate(log10(concentration))
# ... with another name ...
plate_data %>% mutate(log_conc = log10(concentration))
# ... or in place
plate_data %>% mutate(concentration = log10(concentration))
# with subsequent selection
plate_data %>% transmute(log_conc = log10(concentration))
# multiple columns can be specified at the same time
plate_data %>% transmute(log_conc = log10(concentration), 
                         sqr_conc = concentration ** 2)
```

There are many useful functions in `dplyr` to add cumulative aggregates
and ranks to the data. Have a look on this [cheat
sheet](https://github.com/rstudio/cheatsheets/raw/master/data-transformation.pdf).

``` r
# cumulative sum 
plate_data %>% filter(!is.na(intensity)) %>% mutate(cumsum(concentration))
# break the input vector into N buckets based on rank
plate_data %>% filter(!is.na(intensity)) %>% mutate(bucket_no = ntile(intensity, 10))
# break the input vector continuously on rank
plate_data %>% filter(!is.na(intensity)) %>% mutate(percent_rank(intensity))
```

An interesting application is to replace values or create variables
based on a combination of logical evaluations.

``` r
# replace an annoying value with NA
plate_data %>% mutate(concentration = na_if(concentration, 0))
# replace all missing values with a value
plate_data %>% mutate(replace_na(intensity, 0))

# general replacements
plate_data %>% 
  mutate(sample_id = recode(sample_id, control = "lemon", 
                            replicate_A = "banana",
                            replicate_B = "apple"))

# replace based on logical conditions
plate_data %>% filter(!is.na(intensity)) %>% 
  mutate(intensity = if_else(concentration == 100, intensity * 1e4, intensity))

# replace based on multiple conditions ...
plate_data %>% filter(!is.na(intensity)) %>% 
  mutate(intensity = case_when(
    concentration == 0   ~ intensity + 2e2,
    concentration == 1e2 ~ intensity * 1e4,
    # all other cases
    TRUE ~ intensity
  ))

# ... especially useful when you want to create a new variable based on a
# complex combination of existing variables
dplyr::starwars %>%
  select(name:mass, gender, species) %>%
  mutate(type = case_when(
    height > 200 | mass > 200 ~ "large",
    species == "Droid"        ~ "robot",
    TRUE                      ~ "other"
  ))
```

### New Variables from Non-Vectorized Functions

## Hands-On Exercise

1.  Change `"rep.A"` and `"rep.B"` into `"replicate_3"` and
    `"replicate_4"` respectively, arrange by `sample_id` and
    `replicate_id` (descending), then store the result as `plate_data`.

2.  Subtract `control` from all treatments (without modifying the data
    set).