Hands-on - Exploring data with R
================
Dr. Marc A.T. Teunis; Dr. Jan-Willem Lankhaar
12 September 2019

*From R. A. Fisher, 1938: To consult the statistician after an
experiment is finished is often merely to ask him to conduct a post
mortem examination. He can perhaps say what the experiment died of*

## Attribution

This work can be reproduced under licence CC BY-NC, assuming you give
attribution to the original creators. See authors of this document and
the ‘resources’ document in the Github repo for more details.

## Loading the packages used in this demo

``` r
library(tidyverse)
library(gganimate)
library(dslabs)
library(lubridate)
library(ggplot2)
library(gapminder)
library(gganimate)
library(gifski)
library(tidyverse)
```

## Measles example

Every data cycle starts with a question / problem:

  - Problem
  - Plan
  - Data
  - Analysis
  - Conclusion

<!-- end list -->

``` r
 DiagrammeR::grViz("
digraph rmarkdown {
  Problem -> Plan
  Plan -> Data
  Data -> Analysis
  Analysis -> Conclusion 
  Conclusion -> Problem
}
")
```

<!--html_preserve-->

<div id="htmlwidget-e997a070640d9faa459c" class="grViz html-widget" style="width:480px;height:336px;">

</div>

<script type="application/json" data-for="htmlwidget-e997a070640d9faa459c">{"x":{"diagram":"\ndigraph rmarkdown {\n  Problem -> Plan\n  Plan -> Data\n  Data -> Analysis\n  Analysis -> Conclusion \n  Conclusion -> Problem\n}\n","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script>

<!--/html_preserve-->

## Was the introduction of the vaccination against the measles effective in the USA?

``` r
library(dslabs)
data(us_contagious_diseases)
the_disease <- "Measles"
dat <- us_contagious_diseases %>%
  dplyr::filter(!state%in%c("Hawaii","Alaska") &  
      disease == the_disease) %>%
  mutate(rate = count / population * 10000 * 52 / weeks_reporting) 

jet.colors <- colorRampPalette(c("#F0FFFF", "cyan", "#007FFF", "yellow", "#FFBF00", "orange", "red", "#7F0000"), bias = 2.25)

dat %>% mutate(state = reorder(state, desc(state))) %>%
  ggplot(aes(year, state, fill = rate)) +
  geom_tile(color = "white", size = 0.35) +
  scale_x_continuous(expand = c(0,0)) +
  scale_fill_gradientn(colors = jet.colors(16), na.value = 'white') +
  geom_vline(xintercept = 1963, col = "black") +
  theme_minimal() + 
  theme(panel.grid = element_blank()) +
  coord_cartesian(clip = 'off') +
  ggtitle(the_disease) +
  ylab("") +
  xlab("") +  
  theme(legend.position = "bottom", text = element_text(size = 8)) + 
  annotate(geom = "text", x = 1963, y = 50.5, label = "Vaccine introduced", size = 3, hjust = 0)
```

![](demo_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## Build upon existing code

Try changing the line in the code chunk above from

`the_disease <- "Measles"` (*Mazelen* in Dutch) to

`the_disease <- "Smallpox"` (*Pokken* in Dutch) and rerun the code.

Can you find out which part of the code above plots the vertical line
and prints the words “Vaccine introduced”. Try to adapt the figure for
the Smallpox data: The introduction of the smallpox vaccine was already
as early as in 1940. **TIP** You will need to change the year 1963 twice
to 1940 in order for the new figure to be correct for the Smallpox
subset of the data. Try adapting the code and rerun the code chunk.

Question: When did vaccination against smallpox stop? Try googling for
the answer\!

## Exploring the Gapminder dataset in R

The gapminder dataset contains a number of measurements on health and
income outcomes for 184 countries from 1960 to 2016. It also includes
two character vectors, OECD and OPEC, with the names of OECD and OPEC
countries from 2016.

## Inspecting the dataset with R

``` r
data("gapminder", package = "dslabs")
head(gapminder)
```

    ##               country year infant_mortality life_expectancy fertility
    ## 1             Albania 1960           115.40           62.87      6.19
    ## 2             Algeria 1960           148.20           47.50      7.65
    ## 3              Angola 1960           208.00           35.98      7.32
    ## 4 Antigua and Barbuda 1960               NA           62.97      4.43
    ## 5           Argentina 1960            59.87           65.39      3.11
    ## 6             Armenia 1960               NA           66.86      4.55
    ##   population          gdp continent          region
    ## 1    1636054           NA    Europe Southern Europe
    ## 2   11124892  13828152297    Africa Northern Africa
    ## 3    5270844           NA    Africa   Middle Africa
    ## 4      54681           NA  Americas       Caribbean
    ## 5   20619075 108322326649  Americas   South America
    ## 6    1867396           NA      Asia    Western Asia

``` r
gapminder <- gapminder %>% as_tibble()
names(gapminder)
```

    ## [1] "country"          "year"             "infant_mortality"
    ## [4] "life_expectancy"  "fertility"        "population"      
    ## [7] "gdp"              "continent"        "region"

## A very simple example to start with

``` r
gapminder %>% 
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point()
```

![](demo_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

# Fixing overplotting

We call this overlay of points ‘overplotting’.

This can be fixed in several ways:

  - Reducing the transparency of data points  
  - Mapping colour to a variable (continuous or categorical)
  - Reduce the data in the plot
  - Mapping a shape to a variable
  - Add noise (`"jitter"`) to points
  - Choosing different `geom_`
  - Facetting; create panels for ‘categorical’ or so-called ‘factor’
    variables in R
  - Summarize the data
  - Displaying a model / relationship that represents the data (and not
    sho the actual data itself)
  - Or any combination of the above strategies

**Basically you map an `aesthetic` (`aes()`) or a `characteristic` to a
variable**

Let’s go over a few of these overplotting methods

### 1\. Overplotting: Reducing transparency (`alpha`) of points or lines in the data

``` r
gapminder %>% 
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point(alpha = 0.1)
```

![](demo_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

### 2\. Mapping colour to a variable (continuous or categorical)

``` r
gapminder %>% 
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point(aes(colour = continent))
```

![](demo_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

#### or combined with `alpha`

``` r
gapminder %>% 
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point(aes(colour = continent), alpha = 0.1) +
  guides(colour = guide_legend(override.aes = list(alpha = 1)))
```

![](demo_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

#### Choose a different `geom_`

``` r
# install.packages("ggpointdensity")
library(ggpointdensity)

gapminder %>% 
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  stat_density2d(aes(fill = continent), geom = 'polygon', alpha = 0.5) +
  guides(fill = guide_legend(override.aes = list(alpha = 1))) 
```

![](demo_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

#### **Do it yourself:**

  - Try adjusting some of the arguments in the previous `ggplot2` call.
    For example, adjust the `alpha = ...` or change the variable in `x =
    ...`, `y = ...` or `colour = ...`
  - `names(gapminder)` gives you the variable names that you can change
  - Show and discuss the resulting plot with your neighbour
  - What do you think this part does:

`guides(... = guide_legend(override.aes = list(alpha = 1)))`

  - Try to find out by disabling with `#`

### 3\. Reduce the data in the plot

``` r
reduce_data_plot <- gapminder %>% 
  filter(continent == "Africa" | continent == "Europe") %>%
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point(aes(colour = continent), alpha = 0.2) +
  ## override the alpha setting for the points in the legend:
  guides(colour = guide_legend(override.aes = list(alpha = 1))) 

reduce_data_plot
```

![](demo_files/figure-gfm/unnamed-chunk-10-1.png)<!-- -->

### **Discuss with you neighbour:**

  - What does the the `aes()` part of the `geom_point()` do?

### 4\. Mapping a shape to a variable

``` r
## or e.g. show only two years and map a shape to continent
shape_plot <- gapminder %>% 
  dplyr::filter(continent == "Africa" | continent == "Europe",
         year == "1960" | year == "2010") %>%
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point(aes(colour = as_factor(as.character(year)), 
                 shape = continent), 
             alpha = 0.7)

shape_plot
```

![](demo_files/figure-gfm/unnamed-chunk-11-1.png)<!-- -->

### 5\. Facetting

Create panels for ‘categorical’ or so-called ‘factor’ variables in R

``` r
facets_plot <- gapminder %>% 
  dplyr::filter(continent == "Africa" | continent == "Europe",
         year == "1960" | year == "2010") %>%
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point(aes(colour = continent), alpha = 0.5) +
  facet_wrap(~ year)

facets_plot
```

![](demo_files/figure-gfm/unnamed-chunk-12-1.png)<!-- -->

### 6\. Summarize the data

``` r
library(ggrepel)

years <- c("1960", "1970", "1980", "1990", "2000", "2010")

summarize_plot <- gapminder %>% 
  dplyr::filter(year %in% years) %>%
  group_by(continent, year) %>%
  summarise(mean_life_expectancy = mean(life_expectancy),
            mean_fertility = mean(fertility)) %>%
  ggplot(aes(x = mean_fertility,
             y = mean_life_expectancy)) +
  geom_point(aes(colour = continent), alpha = 0.7) 

summarize_plot
```

![](demo_files/figure-gfm/unnamed-chunk-13-1.png)<!-- -->

#### Adding labels to points with `{ggrepel}`

``` r
library(ggrepel)

years <- c("1960", "1970", "1980", "1990", "2000", "2010")

labels_plot <- gapminder %>% 
  dplyr::filter(year %in% years) %>%
  group_by(continent, year) %>%
  summarise(mean_life_expectancy = mean(life_expectancy),
            mean_fertility = mean(fertility)) %>%
  ggplot(aes(x = mean_fertility,
             y = mean_life_expectancy)) +
  geom_point(aes(colour = continent), alpha = 0.7) +
  geom_label_repel(aes(label=year), size = 2.5, box.padding = .5)
  

labels_plot
```

![](demo_files/figure-gfm/unnamed-chunk-14-1.png)<!-- -->

### 7\. Displaying a model

``` r
## Model
lm <- gapminder %>% lm(formula = life_expectancy ~ fertility)

correlation <- cor.test(x = gapminder$fertility, y = gapminder$life_expectancy, method = "pearson")

# save predictions of the model in the new data frame 
# together with variable you want to plot against
predicted_df <- data.frame(gapminder_pred = predict(lm, gapminder), 
                           fertility = gapminder$fertility)
```

#### Add model to plot

``` r
model_plot <- gapminder %>% 
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point(alpha = 0.03) +
  geom_line(data = predicted_df, aes(x = fertility, 
                                     y = gapminder_pred),
            colour = "darkred", size = 1)
model_plot
```

![](demo_files/figure-gfm/unnamed-chunk-16-1.png)<!-- -->

#### Plotting statistics to the graph with the `{ggpubr}` package

![](demo_files/figure-gfm/unnamed-chunk-17-1.png)<!-- -->

#### Using a smoother `geom_smooth` to display potential relationships

``` r
gapminder %>% 
  ggplot(aes(x = fertility,
             y = life_expectancy)) +
  geom_point(alpha = 0.02) +
  geom_smooth(method = "lm") +
  stat_cor(method = "pearson", label.x = 2, label.y = 30) +
  theme_bw()
```

![](demo_files/figure-gfm/unnamed-chunk-18-1.png)<!-- -->

# Time series

So far we have been mapping colours and shapes to categorical variables.
You can also map to continuous variables though.

``` r
continuous <- gapminder %>%
  dplyr::filter(country == "Netherlands" | 
                country == "China" |
                country == "India") %>%
  dplyr::filter(year %in% years) %>%
  ggplot(aes(x = year,
         y = life_expectancy)) +
  geom_point(aes(size = population, colour = country)) +
  guides(colour = guide_legend(override.aes = list(alpha = 1))) +
  geom_line(aes(group = country)) +
  theme_bw()

continuous
```

![](demo_files/figure-gfm/unnamed-chunk-19-1.png)<!-- -->

## Summarize per continent and sum population

``` r
population_plot <- gapminder %>% 
  dplyr::group_by(continent, year) %>%
  dplyr::filter(year %in% years) %>%
  summarise(sum_population = sum(population)) %>% 
  ggplot(aes(x = year, 
             y = sum_population)) +
    geom_point(aes(colour = continent)) +
    geom_line(aes(group = continent,
                  colour = continent))
population_plot
```

![](demo_files/figure-gfm/unnamed-chunk-20-1.png)<!-- -->

# Filtering for one country

``` r
dslabs::gapminder %>%
  dplyr::filter(country == "Rwanda") %>%
  ggplot(aes(x = year, y = life_expectancy)) +
  geom_point() +
  geom_line() +
  ggtitle("Life expectancy for Rwanda over the years") 
```

![](demo_files/figure-gfm/unnamed-chunk-21-1.png)<!-- -->

# Ranking data

``` r
ranking_plot <- gapminder %>%
  dplyr::filter(continent == "Europe",
                year == 2010) %>%
  ggplot(aes(x = reorder(as_factor(country), population),
             y = log10(population))) +
  geom_point() +
  ylab("log10(Population)") +
  xlab("Country") + 
  coord_flip() +
  geom_point(data = filter(gapminder %>%
  dplyr::filter(continent == "Europe",
                year == 2010), population >= 1e7), colour = "red")

ranking_plot
```

![](demo_files/figure-gfm/unnamed-chunk-22-1.png)<!-- -->

## Grouping

Plot individual datapoints over a group aesthetic. Note the outliers\!\!

``` r
gapminder %>% 
  dplyr::filter(continent == "Americas" | continent == "Africa") %>%
  ggplot(aes(x = year,
             y = life_expectancy)) +
  geom_point(aes(colour = country)) +
  theme(legend.position="none") +
  facet_wrap( ~ continent) +
  theme(legend.position="none") 
```

![](demo_files/figure-gfm/unnamed-chunk-23-1.png)<!-- -->

## Summarizing time series data

``` r
gapminder$continent %>% as_factor() %>% levels()
```

    ## [1] "Africa"   "Americas" "Asia"     "Europe"   "Oceania"

``` r
gapminder %>% 
  dplyr::filter(continent == "Americas" | continent == "Africa") %>%
  group_by(continent, year) %>%
  summarise(mean_life_expectancy = mean(life_expectancy)) %>%
  ggplot(aes(x = year,
             y = mean_life_expectancy)) +
  geom_line(aes(group = continent,
                colour = continent)) +
  theme(axis.text.x = element_text(angle = 90, hjust = 1))
```

![](demo_files/figure-gfm/unnamed-chunk-24-1.png)<!-- -->

# Statistical inference

``` r
df <- gapminder %>% 
  dplyr::filter(continent == "Americas" | continent == "Oceania") %>%
  group_by(continent, year)

model <- aov(data = df, life_expectancy ~ continent * year)
anova(model)
```

    ## Analysis of Variance Table
    ## 
    ## Response: life_expectancy
    ##                  Df Sum Sq Mean Sq  F value Pr(>F)    
    ## continent         1   8982    8982  269.104 <2e-16 ***
    ## year              1  58606   58606 1755.931 <2e-16 ***
    ## continent:year    1      9       9    0.278 0.5981    
    ## Residuals      2732  91183      33                    
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

Some remarks on the above Two-way ANOVA:

  - Repeated measures / multilevel models might be more appropriate here
    (paired / nested designs)
  - We did not perform any check on assumptions
  - We performed our analysis on only part of the data

## One more option: categorical values and “jitter”

Sometimes you have overlapping plots and adding transparency with
`alpha()` or mapping colour to underlying categorical values is not
working because there are simple to many points overlapping

Let’s look at an example

``` r
gapminder %>% 
  dplyr::filter(continent == "Americas" |
                continent == "Africa") %>%
  group_by(continent) %>%
  dplyr::filter(year %in% years) %>%
  ggplot(aes(x = year,
             y = infant_mortality)) +
  geom_point(aes(colour = country)) +
  theme(legend.position="none")
```

![](demo_files/figure-gfm/unnamed-chunk-26-1.png)<!-- -->

In such cases it can be helpfull to add some noise to the points
(`position = "jitter"`) to reduce overlapping. This can be a powerfull
approach, especially when combined with setting `alpha()`

``` r
gapminder %>% 
  dplyr::filter(continent == "Americas" |
                continent == "Africa") %>%
  dplyr::filter(year %in% years) %>%
    group_by(continent) %>%
  ggplot(aes(x = year,
             y = infant_mortality)) +
  geom_point(aes(colour = continent), position = "jitter") 
```

![](demo_files/figure-gfm/unnamed-chunk-27-1.png)<!-- -->

## Bar chart

It would be nice to know what the mean child mortality is for both
continents

``` r
gapminder %>% 
  dplyr::filter(continent == "Americas" |
                continent == "Africa") %>%
  dplyr::filter(year %in% years) %>%
  group_by(continent, year) %>%
  summarise(mean_infant_mortality = mean(infant_mortality, na.rm = TRUE)) %>% 
  ggplot(aes(x = year,
             y = mean_infant_mortality)) +
  geom_col(aes(fill = continent), position = "dodge") 
```

![](demo_files/figure-gfm/unnamed-chunk-28-1.png)<!-- -->

## Adding summary data to an existing plot

Now that we have the mean infant mortality for each year for the two
continents, let’s add that data to the previous dot plot where we used
jitter

``` r
mean_inf_mort <- gapminder %>% 
  dplyr::filter(continent == "Americas" |
                continent == "Africa") %>%
  dplyr::filter(year %in% years) %>%
  group_by(continent, year) %>%
  summarise(mean_infant_mortality = mean(infant_mortality, na.rm = TRUE))

gapminder %>% 
  dplyr::filter(continent == "Americas" |
                continent == "Africa") %>%
  dplyr::filter(year %in% years) %>%
    group_by(continent) %>%
  ggplot(aes(x = year,
             y = infant_mortality)) +
  geom_point(aes(colour = continent), position = "jitter") +

## summary data added to previous 
  geom_line(data = mean_inf_mort, aes(x = year, 
                                      y = mean_infant_mortality, 
                                      colour = continent),  size = 2)
```

![](demo_files/figure-gfm/unnamed-chunk-29-1.png)<!-- -->

## Filter data from a graph

In the figure above we can observe a number of countries in ‘Americas’
continent that have a child mortality that are above the average (over
the years) of ‘Africa’. Which countries are this?

``` r
library(ggiraph)

gapminder$country <- 
  str_replace_all(string = gapminder$country, 
                pattern = "'", 
                replacement = "_")


interactive_inf_mort <- gapminder %>% 
  dplyr::filter(continent == "Americas" |
                continent == "Africa") %>%
  dplyr::filter(year %in% years) %>%
    group_by(region, country) %>%
  ggplot(aes(x = year,
             y = infant_mortality)) +
  
  geom_point_interactive(aes(tooltip = country, colour = region), position = "jitter") +
  
#  geom_point(aes(colour = continent), position = "jitter") +

## summary data added to previous 
 geom_line(data = mean_inf_mort, aes(x = year, 
                                      y = mean_infant_mortality, 
                                      colour = continent, group = continent),  size = 2
            )

interactive_inf_mort
```

![](demo_files/figure-gfm/unnamed-chunk-30-1.png)<!-- -->

``` r
gapminder$country %>% as_factor() %>% levels()
```

    ##   [1] "Albania"                        "Algeria"                       
    ##   [3] "Angola"                         "Antigua and Barbuda"           
    ##   [5] "Argentina"                      "Armenia"                       
    ##   [7] "Aruba"                          "Australia"                     
    ##   [9] "Austria"                        "Azerbaijan"                    
    ##  [11] "Bahamas"                        "Bahrain"                       
    ##  [13] "Bangladesh"                     "Barbados"                      
    ##  [15] "Belarus"                        "Belgium"                       
    ##  [17] "Belize"                         "Benin"                         
    ##  [19] "Bhutan"                         "Bolivia"                       
    ##  [21] "Bosnia and Herzegovina"         "Botswana"                      
    ##  [23] "Brazil"                         "Brunei"                        
    ##  [25] "Bulgaria"                       "Burkina Faso"                  
    ##  [27] "Burundi"                        "Cambodia"                      
    ##  [29] "Cameroon"                       "Canada"                        
    ##  [31] "Cape Verde"                     "Central African Republic"      
    ##  [33] "Chad"                           "Chile"                         
    ##  [35] "China"                          "Colombia"                      
    ##  [37] "Comoros"                        "Congo, Dem. Rep."              
    ##  [39] "Congo, Rep."                    "Costa Rica"                    
    ##  [41] "Cote d_Ivoire"                  "Croatia"                       
    ##  [43] "Cuba"                           "Cyprus"                        
    ##  [45] "Czech Republic"                 "Denmark"                       
    ##  [47] "Djibouti"                       "Dominican Republic"            
    ##  [49] "Ecuador"                        "Egypt"                         
    ##  [51] "El Salvador"                    "Equatorial Guinea"             
    ##  [53] "Eritrea"                        "Estonia"                       
    ##  [55] "Ethiopia"                       "Fiji"                          
    ##  [57] "Finland"                        "France"                        
    ##  [59] "French Polynesia"               "Gabon"                         
    ##  [61] "Gambia"                         "Georgia"                       
    ##  [63] "Germany"                        "Ghana"                         
    ##  [65] "Greece"                         "Greenland"                     
    ##  [67] "Grenada"                        "Guatemala"                     
    ##  [69] "Guinea"                         "Guinea-Bissau"                 
    ##  [71] "Guyana"                         "Haiti"                         
    ##  [73] "Honduras"                       "Hong Kong, China"              
    ##  [75] "Hungary"                        "Iceland"                       
    ##  [77] "India"                          "Indonesia"                     
    ##  [79] "Iran"                           "Iraq"                          
    ##  [81] "Ireland"                        "Israel"                        
    ##  [83] "Italy"                          "Jamaica"                       
    ##  [85] "Japan"                          "Jordan"                        
    ##  [87] "Kazakhstan"                     "Kenya"                         
    ##  [89] "Kiribati"                       "South Korea"                   
    ##  [91] "Kuwait"                         "Kyrgyz Republic"               
    ##  [93] "Lao"                            "Latvia"                        
    ##  [95] "Lebanon"                        "Lesotho"                       
    ##  [97] "Liberia"                        "Libya"                         
    ##  [99] "Lithuania"                      "Luxembourg"                    
    ## [101] "Macao, China"                   "Macedonia, FYR"                
    ## [103] "Madagascar"                     "Malawi"                        
    ## [105] "Malaysia"                       "Maldives"                      
    ## [107] "Mali"                           "Malta"                         
    ## [109] "Mauritania"                     "Mauritius"                     
    ## [111] "Mexico"                         "Micronesia, Fed. Sts."         
    ## [113] "Moldova"                        "Mongolia"                      
    ## [115] "Montenegro"                     "Morocco"                       
    ## [117] "Mozambique"                     "Namibia"                       
    ## [119] "Nepal"                          "Netherlands"                   
    ## [121] "New Caledonia"                  "New Zealand"                   
    ## [123] "Nicaragua"                      "Niger"                         
    ## [125] "Nigeria"                        "Norway"                        
    ## [127] "Oman"                           "Pakistan"                      
    ## [129] "Panama"                         "Papua New Guinea"              
    ## [131] "Paraguay"                       "Peru"                          
    ## [133] "Philippines"                    "Poland"                        
    ## [135] "Portugal"                       "Puerto Rico"                   
    ## [137] "Qatar"                          "Romania"                       
    ## [139] "Russia"                         "Rwanda"                        
    ## [141] "St. Lucia"                      "St. Vincent and the Grenadines"
    ## [143] "Samoa"                          "Saudi Arabia"                  
    ## [145] "Senegal"                        "Serbia"                        
    ## [147] "Seychelles"                     "Sierra Leone"                  
    ## [149] "Singapore"                      "Slovak Republic"               
    ## [151] "Slovenia"                       "Solomon Islands"               
    ## [153] "South Africa"                   "Spain"                         
    ## [155] "Sri Lanka"                      "Sudan"                         
    ## [157] "Suriname"                       "Swaziland"                     
    ## [159] "Sweden"                         "Switzerland"                   
    ## [161] "Syria"                          "Tajikistan"                    
    ## [163] "Tanzania"                       "Thailand"                      
    ## [165] "Timor-Leste"                    "Togo"                          
    ## [167] "Tonga"                          "Trinidad and Tobago"           
    ## [169] "Tunisia"                        "Turkey"                        
    ## [171] "Turkmenistan"                   "Uganda"                        
    ## [173] "Ukraine"                        "United Arab Emirates"          
    ## [175] "United Kingdom"                 "United States"                 
    ## [177] "Uruguay"                        "Uzbekistan"                    
    ## [179] "Vanuatu"                        "Venezuela"                     
    ## [181] "West Bank and Gaza"             "Vietnam"                       
    ## [183] "Yemen"                          "Zambia"                        
    ## [185] "Zimbabwe"

``` r
ggiraph(ggobj = interactive_inf_mort)
```

<!--html_preserve-->

<div id="htmlwidget-1edd36fdff1c4005da78" class="girafe html-widget" style="width:480px;height:336px;">

</div>

<script type="application/json" data-for="htmlwidget-1edd36fdff1c4005da78">{"x":{"html":"<?xml version=\"1.0\" encoding=\"UTF-8\"?>\n<svg xmlns=\"http://www.w3.org/2000/svg\" xmlns:xlink=\"http://www.w3.org/1999/xlink\" id=\"svg_2708bd959ee20190912102831\" viewBox=\"0 0 432.00 360.00\">\n  <g>\n    <defs>\n      <clipPath id=\"cl_id27086da15bf620190912102832_1679565025\">\n        <rect x=\"0.00\" y=\"360.00\" width=\"0.00\" height=\"72.00\"/>\n      <\/clipPath>\n    <\/defs>\n    <rect x=\"0.00\" y=\"0.00\" width=\"432.00\" height=\"360.00\" id=\"1\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565025)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke-width=\"0.75\" stroke=\"#FFFFFF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <defs>\n      <clipPath id=\"cl_id27086da15bf620190912102832_1679565026\">\n        <rect x=\"0.00\" y=\"0.00\" width=\"432.00\" height=\"360.00\"/>\n      <\/clipPath>\n    <\/defs>\n    <rect x=\"0.00\" y=\"0.00\" width=\"432.00\" height=\"360.00\" id=\"2\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565026)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke-width=\"1.06698\" stroke=\"#FFFFFF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <defs>\n      <clipPath id=\"cl_id27086da15bf620190912102832_1679565027\">\n        <rect x=\"38.02\" y=\"5.48\" width=\"274.86\" height=\"323.03\"/>\n      <\/clipPath>\n    <\/defs>\n    <rect x=\"38.02\" y=\"5.48\" width=\"274.86\" height=\"323.03\" id=\"3\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <polyline points=\"38.02,288.45 312.89,288.45\" id=\"4\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"0.533489\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,225.29 312.89,225.29\" id=\"5\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"0.533489\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,162.14 312.89,162.14\" id=\"6\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"0.533489\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,98.98 312.89,98.98\" id=\"7\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"0.533489\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,35.82 312.89,35.82\" id=\"8\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"0.533489\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"110.24,328.51 110.24,5.48\" id=\"9\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"0.533489\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"196.79,328.51 196.79,5.48\" id=\"10\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"0.533489\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"283.35,328.51 283.35,5.48\" id=\"11\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"0.533489\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,320.03 312.89,320.03\" id=\"12\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,256.87 312.89,256.87\" id=\"13\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,193.71 312.89,193.71\" id=\"14\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,130.56 312.89,130.56\" id=\"15\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"38.02,67.40 312.89,67.40\" id=\"16\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"66.96,328.51 66.96,5.48\" id=\"17\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"153.51,328.51 153.51,5.48\" id=\"18\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"240.07,328.51 240.07,5.48\" id=\"19\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#EBEBEB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <circle cx=\"70.97\" cy=\"132.83\" r=\"1.47pt\" id=\"20\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"64.18\" cy=\"57.29\" r=\"1.47pt\" id=\"21\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"50.52\" cy=\"244.40\" r=\"1.47pt\" id=\"22\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"64.28\" cy=\"255.60\" r=\"1.47pt\" id=\"23\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"63.31\" cy=\"232.24\" r=\"1.47pt\" id=\"24\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"82.78\" cy=\"83.95\" r=\"1.47pt\" id=\"25\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"71.12\" cy=\"101.00\" r=\"1.47pt\" id=\"26\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"66.61\" cy=\"174.15\" r=\"1.47pt\" id=\"27\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"76.05\" cy=\"156.59\" r=\"1.47pt\" id=\"28\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"78.94\" cy=\"116.30\" r=\"1.47pt\" id=\"29\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"59.89\" cy=\"136.75\" r=\"1.47pt\" id=\"30\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"51.77\" cy=\"109.22\" r=\"1.47pt\" id=\"31\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"59.64\" cy=\"284.91\" r=\"1.47pt\" id=\"32\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"72.50\" cy=\"110.99\" r=\"1.47pt\" id=\"33\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"51.74\" cy=\"158.85\" r=\"1.47pt\" id=\"34\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"71.87\" cy=\"207.24\" r=\"1.47pt\" id=\"35\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"69.21\" cy=\"67.39\" r=\"1.47pt\" id=\"36\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"71.50\" cy=\"100.24\" r=\"1.47pt\" id=\"37\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"51.12\" cy=\"180.33\" r=\"1.47pt\" id=\"38\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"79.58\" cy=\"210.72\" r=\"1.47pt\" id=\"39\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"76.95\" cy=\"56.80\" r=\"1.47pt\" id=\"40\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"54.77\" cy=\"273.28\" r=\"1.47pt\" id=\"41\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"61.74\" cy=\"191.07\" r=\"1.47pt\" id=\"42\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"74.24\" cy=\"166.68\" r=\"1.47pt\" id=\"43\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"57.46\" cy=\"55.27\" r=\"1.47pt\" id=\"44\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"76.84\" cy=\"160.63\" r=\"1.47pt\" id=\"45\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"74.45\" cy=\"115.39\" r=\"1.47pt\" id=\"46\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"52.31\" cy=\"132.58\" r=\"1.47pt\" id=\"47\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"72.85\" cy=\"162.02\" r=\"1.47pt\" id=\"48\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"82.58\" cy=\"135.62\" r=\"1.47pt\" id=\"49\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"57.52\" cy=\"234.51\" r=\"1.47pt\" id=\"50\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"66.74\" cy=\"73.95\" r=\"1.47pt\" id=\"51\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"60.90\" cy=\"142.06\" r=\"1.47pt\" id=\"52\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"64.99\" cy=\"241.70\" r=\"1.47pt\" id=\"53\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"84.17\" cy=\"170.23\" r=\"1.47pt\" id=\"54\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"71.77\" cy=\"139.01\" r=\"1.47pt\" id=\"55\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"52.76\" cy=\"52.25\" r=\"1.47pt\" id=\"56\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"77.83\" cy=\"112.24\" r=\"1.47pt\" id=\"57\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"73.97\" cy=\"178.56\" r=\"1.47pt\" id=\"58\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"78.61\" cy=\"44.40\" r=\"1.47pt\" id=\"59\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"53.33\" cy=\"20.16\" r=\"1.47pt\" id=\"60\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"78.52\" cy=\"149.49\" r=\"1.47pt\" id=\"61\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"71.09\" cy=\"234.38\" r=\"1.47pt\" id=\"62\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"80.04\" cy=\"192.95\" r=\"1.47pt\" id=\"63\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"74.64\" cy=\"136.88\" r=\"1.47pt\" id=\"64\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"53.79\" cy=\"88.88\" r=\"1.47pt\" id=\"65\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"68.17\" cy=\"191.19\" r=\"1.47pt\" id=\"66\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"67.64\" cy=\"150.76\" r=\"1.47pt\" id=\"67\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"66.15\" cy=\"111.60\" r=\"1.47pt\" id=\"68\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"64.52\" cy=\"235.52\" r=\"1.47pt\" id=\"69\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"83.25\" cy=\"240.94\" r=\"1.47pt\" id=\"70\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"52.58\" cy=\"148.38\" r=\"1.47pt\" id=\"71\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"74.04\" cy=\"158.48\" r=\"1.47pt\" id=\"72\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"51.67\" cy=\"160.49\" r=\"1.47pt\" id=\"73\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"75.52\" cy=\"227.07\" r=\"1.47pt\" id=\"74\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"78.06\" cy=\"37.59\" r=\"1.47pt\" id=\"75\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"70.13\" cy=\"184.38\" r=\"1.47pt\" id=\"76\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"62.10\" cy=\"141.03\" r=\"1.47pt\" id=\"77\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"71.31\" cy=\"137.75\" r=\"1.47pt\" id=\"78\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"75.42\" cy=\"114.90\" r=\"1.47pt\" id=\"79\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"53.74\" cy=\"248.53\" r=\"1.47pt\" id=\"80\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"80.09\" cy=\"101.01\" r=\"1.47pt\" id=\"81\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"50.84\" cy=\"152.93\" r=\"1.47pt\" id=\"82\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"68.15\" cy=\"287.33\" r=\"1.47pt\" id=\"83\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"57.37\" cy=\"247.15\" r=\"1.47pt\" id=\"84\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"68.15\" cy=\"244.36\" r=\"1.47pt\" id=\"85\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"54.88\" cy=\"164.41\" r=\"1.47pt\" id=\"86\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"55.40\" cy=\"203.07\" r=\"1.47pt\" id=\"87\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"114.35\" cy=\"135.62\" r=\"1.47pt\" id=\"88\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"114.70\" cy=\"92.68\" r=\"1.47pt\" id=\"89\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"125.16\" cy=\"244.87\" r=\"1.47pt\" id=\"90\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"114.67\" cy=\"287.18\" r=\"1.47pt\" id=\"91\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"111.06\" cy=\"268.12\" r=\"1.47pt\" id=\"92\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"100.55\" cy=\"231.24\" r=\"1.47pt\" id=\"93\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"109.91\" cy=\"121.60\" r=\"1.47pt\" id=\"94\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"120.82\" cy=\"137.62\" r=\"1.47pt\" id=\"95\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"100.27\" cy=\"212.29\" r=\"1.47pt\" id=\"96\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"125.97\" cy=\"190.55\" r=\"1.47pt\" id=\"97\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"121.87\" cy=\"131.43\" r=\"1.47pt\" id=\"98\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"103.32\" cy=\"135.10\" r=\"1.47pt\" id=\"99\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"127.35\" cy=\"160.63\" r=\"1.47pt\" id=\"100\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"103.63\" cy=\"296.66\" r=\"1.47pt\" id=\"101\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"125.75\" cy=\"178.06\" r=\"1.47pt\" id=\"102\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"117.98\" cy=\"146.96\" r=\"1.47pt\" id=\"103\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"100.61\" cy=\"148.37\" r=\"1.47pt\" id=\"104\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"95.09\" cy=\"234.63\" r=\"1.47pt\" id=\"105\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"103.29\" cy=\"231.10\" r=\"1.47pt\" id=\"106\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"107.89\" cy=\"128.66\" r=\"1.47pt\" id=\"107\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"102.99\" cy=\"131.81\" r=\"1.47pt\" id=\"108\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"106.97\" cy=\"208.24\" r=\"1.47pt\" id=\"109\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"114.20\" cy=\"242.73\" r=\"1.47pt\" id=\"110\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"120.33\" cy=\"116.66\" r=\"1.47pt\" id=\"111\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"112.41\" cy=\"272.80\" r=\"1.47pt\" id=\"112\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"97.69\" cy=\"211.78\" r=\"1.47pt\" id=\"113\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"122.32\" cy=\"198.76\" r=\"1.47pt\" id=\"114\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"109.44\" cy=\"115.41\" r=\"1.47pt\" id=\"115\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"111.22\" cy=\"185.24\" r=\"1.47pt\" id=\"116\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"95.00\" cy=\"159.22\" r=\"1.47pt\" id=\"117\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"118.78\" cy=\"140.03\" r=\"1.47pt\" id=\"118\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"120.97\" cy=\"160.86\" r=\"1.47pt\" id=\"119\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"93.89\" cy=\"168.32\" r=\"1.47pt\" id=\"120\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"104.10\" cy=\"170.73\" r=\"1.47pt\" id=\"121\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"117.25\" cy=\"73.46\" r=\"1.47pt\" id=\"122\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"94.62\" cy=\"250.04\" r=\"1.47pt\" id=\"123\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"111.22\" cy=\"113.63\" r=\"1.47pt\" id=\"124\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"125.21\" cy=\"190.95\" r=\"1.47pt\" id=\"125\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"127.26\" cy=\"263.43\" r=\"1.47pt\" id=\"126\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"117.86\" cy=\"204.69\" r=\"1.47pt\" id=\"127\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"126.84\" cy=\"153.80\" r=\"1.47pt\" id=\"128\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"121.80\" cy=\"78.38\" r=\"1.47pt\" id=\"129\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"125.86\" cy=\"198.76\" r=\"1.47pt\" id=\"130\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"123.50\" cy=\"202.32\" r=\"1.47pt\" id=\"131\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"125.89\" cy=\"57.67\" r=\"1.47pt\" id=\"132\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"102.39\" cy=\"72.82\" r=\"1.47pt\" id=\"133\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"119.28\" cy=\"182.98\" r=\"1.47pt\" id=\"134\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"117.65\" cy=\"243.87\" r=\"1.47pt\" id=\"135\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"93.87\" cy=\"222.14\" r=\"1.47pt\" id=\"136\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"122.32\" cy=\"167.43\" r=\"1.47pt\" id=\"137\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"102.98\" cy=\"90.27\" r=\"1.47pt\" id=\"138\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"108.61\" cy=\"240.97\" r=\"1.47pt\" id=\"139\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"93.41\" cy=\"172.74\" r=\"1.47pt\" id=\"140\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"114.75\" cy=\"146.23\" r=\"1.47pt\" id=\"141\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"120.06\" cy=\"106.69\" r=\"1.47pt\" id=\"142\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"100.14\" cy=\"255.11\" r=\"1.47pt\" id=\"143\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"94.38\" cy=\"246.78\" r=\"1.47pt\" id=\"144\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"114.18\" cy=\"189.41\" r=\"1.47pt\" id=\"145\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"126.36\" cy=\"156.58\" r=\"1.47pt\" id=\"146\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"110.35\" cy=\"247.38\" r=\"1.47pt\" id=\"147\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"109.41\" cy=\"246.02\" r=\"1.47pt\" id=\"148\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"96.74\" cy=\"166.31\" r=\"1.47pt\" id=\"149\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"103.44\" cy=\"251.70\" r=\"1.47pt\" id=\"150\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"119.92\" cy=\"78.77\" r=\"1.47pt\" id=\"151\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"94.10\" cy=\"200.40\" r=\"1.47pt\" id=\"152\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"105.47\" cy=\"169.33\" r=\"1.47pt\" id=\"153\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"119.84\" cy=\"158.85\" r=\"1.47pt\" id=\"154\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"103.65\" cy=\"152.27\" r=\"1.47pt\" id=\"155\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"111.96\" cy=\"264.57\" r=\"1.47pt\" id=\"156\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"116.14\" cy=\"165.66\" r=\"1.47pt\" id=\"157\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"109.18\" cy=\"176.65\" r=\"1.47pt\" id=\"158\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"106.89\" cy=\"294.91\" r=\"1.47pt\" id=\"159\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"118.09\" cy=\"258.64\" r=\"1.47pt\" id=\"160\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"104.09\" cy=\"259.28\" r=\"1.47pt\" id=\"161\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"116.87\" cy=\"181.97\" r=\"1.47pt\" id=\"162\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"94.68\" cy=\"228.57\" r=\"1.47pt\" id=\"163\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"157.50\" cy=\"192.33\" r=\"1.47pt\" id=\"164\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"170.46\" cy=\"145.33\" r=\"1.47pt\" id=\"165\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"166.81\" cy=\"273.04\" r=\"1.47pt\" id=\"166\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"138.06\" cy=\"288.19\" r=\"1.47pt\" id=\"167\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"164.03\" cy=\"289.97\" r=\"1.47pt\" id=\"168\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"153.05\" cy=\"256.23\" r=\"1.47pt\" id=\"169\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"157.62\" cy=\"157.46\" r=\"1.47pt\" id=\"170\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"156.98\" cy=\"178.17\" r=\"1.47pt\" id=\"171\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"169.04\" cy=\"250.95\" r=\"1.47pt\" id=\"172\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"162.02\" cy=\"224.16\" r=\"1.47pt\" id=\"173\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"144.46\" cy=\"171.47\" r=\"1.47pt\" id=\"174\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"140.14\" cy=\"154.44\" r=\"1.47pt\" id=\"175\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"162.32\" cy=\"182.10\" r=\"1.47pt\" id=\"176\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"158.81\" cy=\"307.03\" r=\"1.47pt\" id=\"177\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"161.30\" cy=\"240.95\" r=\"1.47pt\" id=\"178\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"152.16\" cy=\"170.46\" r=\"1.47pt\" id=\"179\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"141.49\" cy=\"160.12\" r=\"1.47pt\" id=\"180\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"141.22\" cy=\"284.40\" r=\"1.47pt\" id=\"181\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"151.59\" cy=\"263.71\" r=\"1.47pt\" id=\"182\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"164.51\" cy=\"169.71\" r=\"1.47pt\" id=\"183\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"138.44\" cy=\"153.29\" r=\"1.47pt\" id=\"184\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"141.88\" cy=\"230.10\" r=\"1.47pt\" id=\"185\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"159.83\" cy=\"295.02\" r=\"1.47pt\" id=\"186\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"141.92\" cy=\"176.40\" r=\"1.47pt\" id=\"187\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"168.82\" cy=\"298.94\" r=\"1.47pt\" id=\"188\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"155.50\" cy=\"170.84\" r=\"1.47pt\" id=\"189\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"145.08\" cy=\"239.20\" r=\"1.47pt\" id=\"190\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"161.56\" cy=\"234.02\" r=\"1.47pt\" id=\"191\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"163.00\" cy=\"175.66\" r=\"1.47pt\" id=\"192\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"151.79\" cy=\"220.50\" r=\"1.47pt\" id=\"193\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"145.99\" cy=\"175.76\" r=\"1.47pt\" id=\"194\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"142.19\" cy=\"139.64\" r=\"1.47pt\" id=\"195\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"165.09\" cy=\"226.32\" r=\"1.47pt\" id=\"196\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"157.02\" cy=\"189.68\" r=\"1.47pt\" id=\"197\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"165.73\" cy=\"192.59\" r=\"1.47pt\" id=\"198\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"163.05\" cy=\"209.26\" r=\"1.47pt\" id=\"199\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"168.35\" cy=\"105.04\" r=\"1.47pt\" id=\"200\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"146.41\" cy=\"152.66\" r=\"1.47pt\" id=\"201\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"161.43\" cy=\"252.84\" r=\"1.47pt\" id=\"202\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"169.99\" cy=\"156.57\" r=\"1.47pt\" id=\"203\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"151.17\" cy=\"232.61\" r=\"1.47pt\" id=\"204\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"148.04\" cy=\"278.47\" r=\"1.47pt\" id=\"205\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"141.19\" cy=\"232.36\" r=\"1.47pt\" id=\"206\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"156.40\" cy=\"201.92\" r=\"1.47pt\" id=\"207\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"143.79\" cy=\"116.92\" r=\"1.47pt\" id=\"208\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"140.28\" cy=\"248.02\" r=\"1.47pt\" id=\"209\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"159.30\" cy=\"186.89\" r=\"1.47pt\" id=\"210\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"154.97\" cy=\"128.27\" r=\"1.47pt\" id=\"211\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"136.41\" cy=\"116.41\" r=\"1.47pt\" id=\"212\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"152.35\" cy=\"198.38\" r=\"1.47pt\" id=\"213\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"138.81\" cy=\"279.22\" r=\"1.47pt\" id=\"214\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"170.26\" cy=\"249.17\" r=\"1.47pt\" id=\"215\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"144.95\" cy=\"201.29\" r=\"1.47pt\" id=\"216\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"168.67\" cy=\"99.47\" r=\"1.47pt\" id=\"217\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"153.71\" cy=\"240.21\" r=\"1.47pt\" id=\"218\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"149.21\" cy=\"221.77\" r=\"1.47pt\" id=\"219\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"138.20\" cy=\"149.75\" r=\"1.47pt\" id=\"220\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"168.73\" cy=\"159.61\" r=\"1.47pt\" id=\"221\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"156.96\" cy=\"274.93\" r=\"1.47pt\" id=\"222\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"168.70\" cy=\"257.64\" r=\"1.47pt\" id=\"223\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"147.47\" cy=\"215.94\" r=\"1.47pt\" id=\"224\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"169.01\" cy=\"156.96\" r=\"1.47pt\" id=\"225\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"169.59\" cy=\"286.30\" r=\"1.47pt\" id=\"226\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"155.99\" cy=\"262.92\" r=\"1.47pt\" id=\"227\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"154.18\" cy=\"204.20\" r=\"1.47pt\" id=\"228\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"162.87\" cy=\"288.08\" r=\"1.47pt\" id=\"229\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"137.53\" cy=\"108.69\" r=\"1.47pt\" id=\"230\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"164.31\" cy=\"234.14\" r=\"1.47pt\" id=\"231\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"136.99\" cy=\"209.01\" r=\"1.47pt\" id=\"232\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"144.15\" cy=\"269.50\" r=\"1.47pt\" id=\"233\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"157.52\" cy=\"214.17\" r=\"1.47pt\" id=\"234\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"157.79\" cy=\"182.47\" r=\"1.47pt\" id=\"235\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"149.11\" cy=\"185.00\" r=\"1.47pt\" id=\"236\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"140.15\" cy=\"276.71\" r=\"1.47pt\" id=\"237\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"162.39\" cy=\"232.61\" r=\"1.47pt\" id=\"238\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"142.72\" cy=\"158.73\" r=\"1.47pt\" id=\"239\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"137.51\" cy=\"304.11\" r=\"1.47pt\" id=\"240\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"149.73\" cy=\"275.19\" r=\"1.47pt\" id=\"241\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"143.23\" cy=\"275.57\" r=\"1.47pt\" id=\"242\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"137.82\" cy=\"198.64\" r=\"1.47pt\" id=\"243\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"162.74\" cy=\"236.15\" r=\"1.47pt\" id=\"244\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"185.98\" cy=\"269.87\" r=\"1.47pt\" id=\"245\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"193.11\" cy=\"151.39\" r=\"1.47pt\" id=\"246\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"184.31\" cy=\"289.73\" r=\"1.47pt\" id=\"247\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"201.92\" cy=\"289.22\" r=\"1.47pt\" id=\"248\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"190.03\" cy=\"295.15\" r=\"1.47pt\" id=\"249\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"191.95\" cy=\"299.83\" r=\"1.47pt\" id=\"250\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"196.02\" cy=\"279.34\" r=\"1.47pt\" id=\"251\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"206.40\" cy=\"183.60\" r=\"1.47pt\" id=\"252\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"203.80\" cy=\"211.91\" r=\"1.47pt\" id=\"253\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"187.36\" cy=\"267.22\" r=\"1.47pt\" id=\"254\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"190.19\" cy=\"255.73\" r=\"1.47pt\" id=\"255\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"206.51\" cy=\"190.56\" r=\"1.47pt\" id=\"256\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"192.64\" cy=\"188.78\" r=\"1.47pt\" id=\"257\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"198.66\" cy=\"211.91\" r=\"1.47pt\" id=\"258\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"212.88\" cy=\"311.43\" r=\"1.47pt\" id=\"259\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"201.12\" cy=\"259.14\" r=\"1.47pt\" id=\"260\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"188.78\" cy=\"174.39\" r=\"1.47pt\" id=\"261\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"203.00\" cy=\"173.76\" r=\"1.47pt\" id=\"262\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"211.23\" cy=\"299.82\" r=\"1.47pt\" id=\"263\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"207.92\" cy=\"283.53\" r=\"1.47pt\" id=\"264\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"179.79\" cy=\"208.98\" r=\"1.47pt\" id=\"265\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"201.81\" cy=\"168.70\" r=\"1.47pt\" id=\"266\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"186.38\" cy=\"243.10\" r=\"1.47pt\" id=\"267\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"211.77\" cy=\"301.96\" r=\"1.47pt\" id=\"268\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"183.12\" cy=\"187.54\" r=\"1.47pt\" id=\"269\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"205.77\" cy=\"306.63\" r=\"1.47pt\" id=\"270\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"186.35\" cy=\"202.93\" r=\"1.47pt\" id=\"271\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"194.31\" cy=\"261.28\" r=\"1.47pt\" id=\"272\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"195.09\" cy=\"264.19\" r=\"1.47pt\" id=\"273\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"183.94\" cy=\"240.45\" r=\"1.47pt\" id=\"274\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"204.07\" cy=\"262.05\" r=\"1.47pt\" id=\"275\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"210.81\" cy=\"158.49\" r=\"1.47pt\" id=\"276\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"207.30\" cy=\"202.54\" r=\"1.47pt\" id=\"277\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"191.12\" cy=\"166.42\" r=\"1.47pt\" id=\"278\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"210.81\" cy=\"243.60\" r=\"1.47pt\" id=\"279\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"186.17\" cy=\"218.99\" r=\"1.47pt\" id=\"280\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"207.41\" cy=\"219.23\" r=\"1.47pt\" id=\"281\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"187.87\" cy=\"297.30\" r=\"1.47pt\" id=\"282\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"189.81\" cy=\"244.49\" r=\"1.47pt\" id=\"283\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"184.92\" cy=\"142.19\" r=\"1.47pt\" id=\"284\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"211.20\" cy=\"148.86\" r=\"1.47pt\" id=\"285\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"189.40\" cy=\"261.15\" r=\"1.47pt\" id=\"286\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"191.34\" cy=\"192.45\" r=\"1.47pt\" id=\"287\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"192.96\" cy=\"263.07\" r=\"1.47pt\" id=\"288\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"186.44\" cy=\"287.96\" r=\"1.47pt\" id=\"289\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"210.54\" cy=\"236.92\" r=\"1.47pt\" id=\"290\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"185.99\" cy=\"230.60\" r=\"1.47pt\" id=\"291\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"190.83\" cy=\"105.16\" r=\"1.47pt\" id=\"292\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"183.15\" cy=\"275.18\" r=\"1.47pt\" id=\"293\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"184.95\" cy=\"196.11\" r=\"1.47pt\" id=\"294\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"195.87\" cy=\"140.03\" r=\"1.47pt\" id=\"295\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"197.93\" cy=\"155.06\" r=\"1.47pt\" id=\"296\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"188.66\" cy=\"221.63\" r=\"1.47pt\" id=\"297\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"180.93\" cy=\"294.89\" r=\"1.47pt\" id=\"298\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"206.66\" cy=\"273.16\" r=\"1.47pt\" id=\"299\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"204.26\" cy=\"240.33\" r=\"1.47pt\" id=\"300\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"186.40\" cy=\"118.31\" r=\"1.47pt\" id=\"301\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"186.75\" cy=\"257.39\" r=\"1.47pt\" id=\"302\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"191.94\" cy=\"255.73\" r=\"1.47pt\" id=\"303\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"188.77\" cy=\"145.59\" r=\"1.47pt\" id=\"304\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"203.12\" cy=\"160.99\" r=\"1.47pt\" id=\"305\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"187.48\" cy=\"287.56\" r=\"1.47pt\" id=\"306\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"189.69\" cy=\"273.18\" r=\"1.47pt\" id=\"307\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"207.82\" cy=\"248.90\" r=\"1.47pt\" id=\"308\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"199.98\" cy=\"202.29\" r=\"1.47pt\" id=\"309\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"208.86\" cy=\"296.39\" r=\"1.47pt\" id=\"310\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"211.58\" cy=\"294.40\" r=\"1.47pt\" id=\"311\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"180.06\" cy=\"231.23\" r=\"1.47pt\" id=\"312\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"188.11\" cy=\"302.09\" r=\"1.47pt\" id=\"313\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"192.88\" cy=\"122.34\" r=\"1.47pt\" id=\"314\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"186.01\" cy=\"260.16\" r=\"1.47pt\" id=\"315\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"197.84\" cy=\"219.10\" r=\"1.47pt\" id=\"316\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"199.46\" cy=\"268.63\" r=\"1.47pt\" id=\"317\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"194.24\" cy=\"249.30\" r=\"1.47pt\" id=\"318\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"181.77\" cy=\"193.21\" r=\"1.47pt\" id=\"319\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"190.74\" cy=\"206.08\" r=\"1.47pt\" id=\"320\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"181.69\" cy=\"286.04\" r=\"1.47pt\" id=\"321\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"202.34\" cy=\"264.08\" r=\"1.47pt\" id=\"322\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"186.08\" cy=\"179.33\" r=\"1.47pt\" id=\"323\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"199.65\" cy=\"308.15\" r=\"1.47pt\" id=\"324\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"190.55\" cy=\"294.38\" r=\"1.47pt\" id=\"325\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"183.24\" cy=\"288.84\" r=\"1.47pt\" id=\"326\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"206.83\" cy=\"176.92\" r=\"1.47pt\" id=\"327\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"207.04\" cy=\"255.37\" r=\"1.47pt\" id=\"328\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"234.88\" cy=\"277.20\" r=\"1.47pt\" id=\"329\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"249.84\" cy=\"157.97\" r=\"1.47pt\" id=\"330\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"246.87\" cy=\"302.61\" r=\"1.47pt\" id=\"331\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"254.62\" cy=\"297.29\" r=\"1.47pt\" id=\"332\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"236.05\" cy=\"303.61\" r=\"1.47pt\" id=\"333\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"249.37\" cy=\"301.34\" r=\"1.47pt\" id=\"334\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"238.89\" cy=\"293.39\" r=\"1.47pt\" id=\"335\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"224.25\" cy=\"207.23\" r=\"1.47pt\" id=\"336\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"236.10\" cy=\"245.76\" r=\"1.47pt\" id=\"337\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"247.88\" cy=\"253.85\" r=\"1.47pt\" id=\"338\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"242.86\" cy=\"284.55\" r=\"1.47pt\" id=\"339\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"249.67\" cy=\"198.52\" r=\"1.47pt\" id=\"340\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"229.39\" cy=\"202.04\" r=\"1.47pt\" id=\"341\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"245.70\" cy=\"203.94\" r=\"1.47pt\" id=\"342\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"229.31\" cy=\"313.47\" r=\"1.47pt\" id=\"343\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"223.30\" cy=\"283.28\" r=\"1.47pt\" id=\"344\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"230.89\" cy=\"176.53\" r=\"1.47pt\" id=\"345\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"249.20\" cy=\"186.52\" r=\"1.47pt\" id=\"346\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"224.90\" cy=\"308.42\" r=\"1.47pt\" id=\"347\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"233.61\" cy=\"293.24\" r=\"1.47pt\" id=\"348\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"243.20\" cy=\"228.21\" r=\"1.47pt\" id=\"349\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"225.60\" cy=\"184.38\" r=\"1.47pt\" id=\"350\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"226.05\" cy=\"223.27\" r=\"1.47pt\" id=\"351\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"253.05\" cy=\"305.87\" r=\"1.47pt\" id=\"352\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"235.13\" cy=\"194.35\" r=\"1.47pt\" id=\"353\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"242.10\" cy=\"311.83\" r=\"1.47pt\" id=\"354\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"243.53\" cy=\"219.35\" r=\"1.47pt\" id=\"355\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"256.88\" cy=\"277.96\" r=\"1.47pt\" id=\"356\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"238.69\" cy=\"284.15\" r=\"1.47pt\" id=\"357\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"242.45\" cy=\"273.30\" r=\"1.47pt\" id=\"358\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"225.32\" cy=\"286.19\" r=\"1.47pt\" id=\"359\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"232.69\" cy=\"187.65\" r=\"1.47pt\" id=\"360\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"250.07\" cy=\"246.39\" r=\"1.47pt\" id=\"361\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"234.38\" cy=\"206.97\" r=\"1.47pt\" id=\"362\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"252.50\" cy=\"249.81\" r=\"1.47pt\" id=\"363\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"254.95\" cy=\"240.08\" r=\"1.47pt\" id=\"364\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"231.98\" cy=\"238.06\" r=\"1.47pt\" id=\"365\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"242.73\" cy=\"302.86\" r=\"1.47pt\" id=\"366\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"234.97\" cy=\"269.64\" r=\"1.47pt\" id=\"367\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"252.51\" cy=\"189.78\" r=\"1.47pt\" id=\"368\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"238.70\" cy=\"184.99\" r=\"1.47pt\" id=\"369\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"241.92\" cy=\"273.03\" r=\"1.47pt\" id=\"370\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"256.15\" cy=\"225.29\" r=\"1.47pt\" id=\"371\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"248.76\" cy=\"281.51\" r=\"1.47pt\" id=\"372\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"249.28\" cy=\"296.41\" r=\"1.47pt\" id=\"373\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"222.90\" cy=\"236.04\" r=\"1.47pt\" id=\"374\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"236.63\" cy=\"213.80\" r=\"1.47pt\" id=\"375\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"230.03\" cy=\"164.67\" r=\"1.47pt\" id=\"376\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"232.68\" cy=\"289.46\" r=\"1.47pt\" id=\"377\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"240.67\" cy=\"231.99\" r=\"1.47pt\" id=\"378\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"242.25\" cy=\"189.29\" r=\"1.47pt\" id=\"379\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"233.58\" cy=\"173.49\" r=\"1.47pt\" id=\"380\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"254.14\" cy=\"223.77\" r=\"1.47pt\" id=\"381\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"223.67\" cy=\"299.32\" r=\"1.47pt\" id=\"382\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"240.60\" cy=\"292.74\" r=\"1.47pt\" id=\"383\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"254.91\" cy=\"266.72\" r=\"1.47pt\" id=\"384\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"225.21\" cy=\"174.76\" r=\"1.47pt\" id=\"385\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"254.74\" cy=\"257.63\" r=\"1.47pt\" id=\"386\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"237.86\" cy=\"278.86\" r=\"1.47pt\" id=\"387\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"243.02\" cy=\"192.33\" r=\"1.47pt\" id=\"388\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"239.99\" cy=\"178.55\" r=\"1.47pt\" id=\"389\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"248.39\" cy=\"292.37\" r=\"1.47pt\" id=\"390\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"239.88\" cy=\"285.03\" r=\"1.47pt\" id=\"391\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"238.08\" cy=\"282.64\" r=\"1.47pt\" id=\"392\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"250.77\" cy=\"182.09\" r=\"1.47pt\" id=\"393\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"246.93\" cy=\"300.83\" r=\"1.47pt\" id=\"394\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"248.12\" cy=\"295.78\" r=\"1.47pt\" id=\"395\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"256.57\" cy=\"233.49\" r=\"1.47pt\" id=\"396\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"250.21\" cy=\"304.50\" r=\"1.47pt\" id=\"397\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"250.89\" cy=\"139.01\" r=\"1.47pt\" id=\"398\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"242.20\" cy=\"251.80\" r=\"1.47pt\" id=\"399\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"242.32\" cy=\"234.40\" r=\"1.47pt\" id=\"400\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"228.15\" cy=\"281.88\" r=\"1.47pt\" id=\"401\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"242.20\" cy=\"213.94\" r=\"1.47pt\" id=\"402\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"223.77\" cy=\"218.59\" r=\"1.47pt\" id=\"403\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"223.39\" cy=\"223.76\" r=\"1.47pt\" id=\"404\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"252.89\" cy=\"288.08\" r=\"1.47pt\" id=\"405\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"229.48\" cy=\"286.81\" r=\"1.47pt\" id=\"406\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"237.59\" cy=\"206.33\" r=\"1.47pt\" id=\"407\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"226.04\" cy=\"311.06\" r=\"1.47pt\" id=\"408\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"239.86\" cy=\"301.57\" r=\"1.47pt\" id=\"409\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"246.72\" cy=\"296.67\" r=\"1.47pt\" id=\"410\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"224.62\" cy=\"196.73\" r=\"1.47pt\" id=\"411\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"233.24\" cy=\"239.83\" r=\"1.47pt\" id=\"412\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"271.60\" cy=\"290.35\" r=\"1.47pt\" id=\"413\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"270.31\" cy=\"181.59\" r=\"1.47pt\" id=\"414\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"269.39\" cy=\"310.29\" r=\"1.47pt\" id=\"415\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"266.66\" cy=\"303.61\" r=\"1.47pt\" id=\"416\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"293.38\" cy=\"305.64\" r=\"1.47pt\" id=\"417\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"285.29\" cy=\"302.85\" r=\"1.47pt\" id=\"418\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"287.10\" cy=\"299.44\" r=\"1.47pt\" id=\"419\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"275.10\" cy=\"230.35\" r=\"1.47pt\" id=\"420\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"272.62\" cy=\"273.55\" r=\"1.47pt\" id=\"421\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"285.68\" cy=\"269.74\" r=\"1.47pt\" id=\"422\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"288.46\" cy=\"301.33\" r=\"1.47pt\" id=\"423\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"298.01\" cy=\"231.99\" r=\"1.47pt\" id=\"424\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"292.25\" cy=\"239.44\" r=\"1.47pt\" id=\"425\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"295.79\" cy=\"236.41\" r=\"1.47pt\" id=\"426\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"284.18\" cy=\"313.82\" r=\"1.47pt\" id=\"427\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"295.36\" cy=\"290.61\" r=\"1.47pt\" id=\"428\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"283.52\" cy=\"191.55\" r=\"1.47pt\" id=\"429\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"267.70\" cy=\"201.81\" r=\"1.47pt\" id=\"430\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"289.12\" cy=\"310.44\" r=\"1.47pt\" id=\"431\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"284.78\" cy=\"299.95\" r=\"1.47pt\" id=\"432\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"277.05\" cy=\"240.34\" r=\"1.47pt\" id=\"433\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"300.39\" cy=\"212.91\" r=\"1.47pt\" id=\"434\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"289.62\" cy=\"266.71\" r=\"1.47pt\" id=\"435\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"294.70\" cy=\"308.80\" r=\"1.47pt\" id=\"436\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"284.43\" cy=\"222.88\" r=\"1.47pt\" id=\"437\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"284.11\" cy=\"313.83\" r=\"1.47pt\" id=\"438\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"279.21\" cy=\"241.46\" r=\"1.47pt\" id=\"439\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"287.82\" cy=\"284.80\" r=\"1.47pt\" id=\"440\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"288.94\" cy=\"293.12\" r=\"1.47pt\" id=\"441\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"289.99\" cy=\"289.34\" r=\"1.47pt\" id=\"442\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"279.39\" cy=\"298.42\" r=\"1.47pt\" id=\"443\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"283.49\" cy=\"220.37\" r=\"1.47pt\" id=\"444\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"287.15\" cy=\"270.26\" r=\"1.47pt\" id=\"445\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"277.49\" cy=\"255.87\" r=\"1.47pt\" id=\"446\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"267.56\" cy=\"265.97\" r=\"1.47pt\" id=\"447\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"298.34\" cy=\"254.72\" r=\"1.47pt\" id=\"448\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"286.96\" cy=\"256.62\" r=\"1.47pt\" id=\"449\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"294.62\" cy=\"305.00\" r=\"1.47pt\" id=\"450\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"269.95\" cy=\"284.15\" r=\"1.47pt\" id=\"451\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"296.14\" cy=\"230.09\" r=\"1.47pt\" id=\"452\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"296.94\" cy=\"227.30\" r=\"1.47pt\" id=\"453\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"286.66\" cy=\"277.72\" r=\"1.47pt\" id=\"454\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"295.78\" cy=\"212.02\" r=\"1.47pt\" id=\"455\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"272.26\" cy=\"293.89\" r=\"1.47pt\" id=\"456\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"288.13\" cy=\"300.45\" r=\"1.47pt\" id=\"457\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"267.19\" cy=\"266.47\" r=\"1.47pt\" id=\"458\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"280.91\" cy=\"225.05\" r=\"1.47pt\" id=\"459\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"289.46\" cy=\"237.68\" r=\"1.47pt\" id=\"460\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"280.92\" cy=\"301.96\" r=\"1.47pt\" id=\"461\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"271.49\" cy=\"266.86\" r=\"1.47pt\" id=\"462\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"271.53\" cy=\"247.40\" r=\"1.47pt\" id=\"463\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"287.53\" cy=\"215.31\" r=\"1.47pt\" id=\"464\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"272.99\" cy=\"231.47\" r=\"1.47pt\" id=\"465\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"273.71\" cy=\"303.23\" r=\"1.47pt\" id=\"466\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"288.05\" cy=\"301.82\" r=\"1.47pt\" id=\"467\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"268.02\" cy=\"284.02\" r=\"1.47pt\" id=\"468\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"279.76\" cy=\"229.22\" r=\"1.47pt\" id=\"469\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"273.50\" cy=\"272.67\" r=\"1.47pt\" id=\"470\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"293.31\" cy=\"292.25\" r=\"1.47pt\" id=\"471\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"299.89\" cy=\"236.53\" r=\"1.47pt\" id=\"472\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"271.27\" cy=\"217.08\" r=\"1.47pt\" id=\"473\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"281.23\" cy=\"298.55\" r=\"1.47pt\" id=\"474\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"269.38\" cy=\"294.25\" r=\"1.47pt\" id=\"475\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"289.48\" cy=\"299.43\" r=\"1.47pt\" id=\"476\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"268.76\" cy=\"264.69\" r=\"1.47pt\" id=\"477\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"287.31\" cy=\"302.21\" r=\"1.47pt\" id=\"478\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"289.94\" cy=\"296.54\" r=\"1.47pt\" id=\"479\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"291.59\" cy=\"261.05\" r=\"1.47pt\" id=\"480\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"288.77\" cy=\"304.62\" r=\"1.47pt\" id=\"481\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"297.55\" cy=\"184.87\" r=\"1.47pt\" id=\"482\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"272.95\" cy=\"271.77\" r=\"1.47pt\" id=\"483\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"268.20\" cy=\"252.70\" r=\"1.47pt\" id=\"484\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"293.48\" cy=\"292.13\" r=\"1.47pt\" id=\"485\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"271.05\" cy=\"245.37\" r=\"1.47pt\" id=\"486\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"276.97\" cy=\"266.46\" r=\"1.47pt\" id=\"487\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"299.68\" cy=\"245.11\" r=\"1.47pt\" id=\"488\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"278.12\" cy=\"293.51\" r=\"1.47pt\" id=\"489\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"296.17\" cy=\"301.21\" r=\"1.47pt\" id=\"490\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"266.07\" cy=\"257.51\" r=\"1.47pt\" id=\"491\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"267.83\" cy=\"312.07\" r=\"1.47pt\" id=\"492\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"292.07\" cy=\"306.65\" r=\"1.47pt\" id=\"493\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"281.80\" cy=\"301.96\" r=\"1.47pt\" id=\"494\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"286.77\" cy=\"253.21\" r=\"1.47pt\" id=\"495\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <circle cx=\"282.15\" cy=\"249.55\" r=\"1.47pt\" id=\"496\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <polyline points=\"66.96,126.92 110.24,154.33 153.51,185.02 196.79,205.14 240.07,220.33 283.35,249.25\" id=\"497\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"4.26791\" stroke=\"#F8766D\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"66.96,202.48 110.24,229.30 153.51,256.52 196.79,276.78 240.07,289.87 283.35,296.50\" id=\"498\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565027)\" fill=\"none\" stroke-width=\"4.26791\" stroke=\"#DB8E00\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <rect x=\"38.02\" y=\"5.48\" width=\"274.86\" height=\"323.03\" id=\"499\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565027)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <defs>\n      <clipPath id=\"cl_id27086da15bf620190912102832_1679565028\">\n        <rect x=\"0.00\" y=\"0.00\" width=\"432.00\" height=\"360.00\"/>\n      <\/clipPath>\n    <\/defs>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"28.20\" y=\"323.18\" id=\"500\" font-size=\"6.60pt\" fill=\"#4D4D4D\" fill-opacity=\"1\" font-family=\"Arial\">0<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"23.30\" y=\"260.02\" id=\"501\" font-size=\"6.60pt\" fill=\"#4D4D4D\" fill-opacity=\"1\" font-family=\"Arial\">50<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"18.41\" y=\"196.86\" id=\"502\" font-size=\"6.60pt\" fill=\"#4D4D4D\" fill-opacity=\"1\" font-family=\"Arial\">100<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"18.41\" y=\"133.71\" id=\"503\" font-size=\"6.60pt\" fill=\"#4D4D4D\" fill-opacity=\"1\" font-family=\"Arial\">150<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"18.41\" y=\"70.55\" id=\"504\" font-size=\"6.60pt\" fill=\"#4D4D4D\" fill-opacity=\"1\" font-family=\"Arial\">200<\/text>\n    <\/g>\n    <polyline points=\"35.28,320.03 38.02,320.03\" id=\"505\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"35.28,256.87 38.02,256.87\" id=\"506\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"35.28,193.71 38.02,193.71\" id=\"507\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"35.28,130.56 38.02,130.56\" id=\"508\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"35.28,67.40 38.02,67.40\" id=\"509\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"66.96,331.25 66.96,328.51\" id=\"510\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"153.51,331.25 153.51,328.51\" id=\"511\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <polyline points=\"240.07,331.25 240.07,328.51\" id=\"512\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" fill=\"none\" stroke-width=\"1.06698\" stroke=\"#333333\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\"/>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"57.17\" y=\"339.74\" id=\"513\" font-size=\"6.60pt\" fill=\"#4D4D4D\" fill-opacity=\"1\" font-family=\"Arial\">1960<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"143.73\" y=\"339.74\" id=\"514\" font-size=\"6.60pt\" fill=\"#4D4D4D\" fill-opacity=\"1\" font-family=\"Arial\">1980<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"230.28\" y=\"339.74\" id=\"515\" font-size=\"6.60pt\" fill=\"#4D4D4D\" fill-opacity=\"1\" font-family=\"Arial\">2000<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"164.76\" y=\"352.21\" id=\"516\" font-size=\"8.25pt\" font-family=\"Arial\">year<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text transform=\"translate(13.35,204.29) rotate(-90)\" id=\"517\" font-size=\"8.25pt\" font-family=\"Arial\">infant_mortality<\/text>\n    <\/g>\n    <rect x=\"323.85\" y=\"58.64\" width=\"102.67\" height=\"216.71\" id=\"518\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"329.33\" y=\"73.15\" id=\"519\" font-size=\"8.25pt\" font-family=\"Arial\">region<\/text>\n    <\/g>\n    <rect x=\"329.33\" y=\"79.79\" width=\"17.28\" height=\"17.28\" id=\"520\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"88.43\" r=\"1.47pt\" id=\"521\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#F8766D\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#F8766D\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"88.43\" x2=\"344.88\" y2=\"88.43\" id=\"522\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#F8766D\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"97.07\" width=\"17.28\" height=\"17.28\" id=\"523\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"105.71\" r=\"1.47pt\" id=\"524\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#DB8E00\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#DB8E00\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"105.71\" x2=\"344.88\" y2=\"105.71\" id=\"525\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#DB8E00\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"114.35\" width=\"17.28\" height=\"17.28\" id=\"526\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"122.99\" r=\"1.47pt\" id=\"527\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#AEA200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"122.99\" x2=\"344.88\" y2=\"122.99\" id=\"528\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#AEA200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"131.63\" width=\"17.28\" height=\"17.28\" id=\"529\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"140.27\" r=\"1.47pt\" id=\"530\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#64B200\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"140.27\" x2=\"344.88\" y2=\"140.27\" id=\"531\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#64B200\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"148.91\" width=\"17.28\" height=\"17.28\" id=\"532\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"157.55\" r=\"1.47pt\" id=\"533\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#00BD5C\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"157.55\" x2=\"344.88\" y2=\"157.55\" id=\"534\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#00BD5C\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"166.19\" width=\"17.28\" height=\"17.28\" id=\"535\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"174.83\" r=\"1.47pt\" id=\"536\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#00C1A7\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"174.83\" x2=\"344.88\" y2=\"174.83\" id=\"537\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#00C1A7\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"183.47\" width=\"17.28\" height=\"17.28\" id=\"538\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"192.11\" r=\"1.47pt\" id=\"539\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#00BADE\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"192.11\" x2=\"344.88\" y2=\"192.11\" id=\"540\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#00BADE\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"200.75\" width=\"17.28\" height=\"17.28\" id=\"541\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"209.39\" r=\"1.47pt\" id=\"542\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#00A6FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"209.39\" x2=\"344.88\" y2=\"209.39\" id=\"543\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#00A6FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"218.03\" width=\"17.28\" height=\"17.28\" id=\"544\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"226.67\" r=\"1.47pt\" id=\"545\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#B385FF\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"226.67\" x2=\"344.88\" y2=\"226.67\" id=\"546\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#B385FF\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"235.31\" width=\"17.28\" height=\"17.28\" id=\"547\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"243.95\" r=\"1.47pt\" id=\"548\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#EF67EB\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"243.95\" x2=\"344.88\" y2=\"243.95\" id=\"549\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#EF67EB\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <rect x=\"329.33\" y=\"252.59\" width=\"17.28\" height=\"17.28\" id=\"550\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FFFFFF\" fill-opacity=\"1\" stroke=\"none\"/>\n    <circle cx=\"337.97\" cy=\"261.23\" r=\"1.47pt\" id=\"551\" clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\" fill=\"#FF63B6\" fill-opacity=\"1\" stroke-width=\"0.708661\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"round\"/>\n    <line x1=\"331.05\" y1=\"261.23\" x2=\"344.88\" y2=\"261.23\" id=\"552\" clip-path=\"url(#cl_svg_2708bd959ee20190912102831_1679565028)\" stroke-width=\"4.26791\" stroke=\"#FF63B6\" stroke-opacity=\"1\" stroke-linejoin=\"round\" stroke-linecap=\"butt\" fill=\"#FFFFFF\" fill-opacity=\"1\"/>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"91.58\" id=\"553\" font-size=\"6.60pt\" font-family=\"Arial\">Africa<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"108.86\" id=\"554\" font-size=\"6.60pt\" font-family=\"Arial\">Americas<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"126.14\" id=\"555\" font-size=\"6.60pt\" font-family=\"Arial\">Caribbean<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"143.42\" id=\"556\" font-size=\"6.60pt\" font-family=\"Arial\">Central America<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"160.70\" id=\"557\" font-size=\"6.60pt\" font-family=\"Arial\">Eastern Africa<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"177.98\" id=\"558\" font-size=\"6.60pt\" font-family=\"Arial\">Middle Africa<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"195.26\" id=\"559\" font-size=\"6.60pt\" font-family=\"Arial\">Northern Africa<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"212.54\" id=\"560\" font-size=\"6.60pt\" font-family=\"Arial\">Northern America<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"229.82\" id=\"561\" font-size=\"6.60pt\" font-family=\"Arial\">South America<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"247.10\" id=\"562\" font-size=\"6.60pt\" font-family=\"Arial\">Southern Africa<\/text>\n    <\/g>\n    <g clip-path=\"url(#cl_id27086da15bf620190912102832_1679565028)\">\n      <text x=\"352.08\" y=\"264.38\" id=\"563\" font-size=\"6.60pt\" font-family=\"Arial\">Western Africa<\/text>\n    <\/g>\n  <\/g>\n<\/svg>\n","js":"function zzz(){document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('20').setAttribute('title','Algeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('21').setAttribute('title','Angola');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('22').setAttribute('title','Argentina');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('23').setAttribute('title','Bahamas');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('24').setAttribute('title','Barbados');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('25').setAttribute('title','Benin');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('26').setAttribute('title','Bolivia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('27').setAttribute('title','Botswana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('28').setAttribute('title','Brazil');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('29').setAttribute('title','Burkina Faso');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('30').setAttribute('title','Burundi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('31').setAttribute('title','Cameroon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('32').setAttribute('title','Canada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('33').setAttribute('title','Central African Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('34').setAttribute('title','Chile');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('35').setAttribute('title','Colombia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('36').setAttribute('title','Comoros');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('37').setAttribute('title','Congo, Dem. Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('38').setAttribute('title','Congo, Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('39').setAttribute('title','Costa Rica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('40').setAttribute('title','Cote d_Ivoire');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('41').setAttribute('title','Cuba');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('42').setAttribute('title','Dominican Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('43').setAttribute('title','Ecuador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('44').setAttribute('title','Egypt');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('45').setAttribute('title','El Salvador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('46').setAttribute('title','Ethiopia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('47').setAttribute('title','Gambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('48').setAttribute('title','Ghana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('49').setAttribute('title','Guatemala');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('50').setAttribute('title','Guyana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('51').setAttribute('title','Haiti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('52').setAttribute('title','Honduras');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('53').setAttribute('title','Jamaica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('54').setAttribute('title','Kenya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('55').setAttribute('title','Lesotho');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('56').setAttribute('title','Liberia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('57').setAttribute('title','Libya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('58').setAttribute('title','Madagascar');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('59').setAttribute('title','Malawi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('60').setAttribute('title','Mali');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('61').setAttribute('title','Mauritania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('62').setAttribute('title','Mauritius');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('63').setAttribute('title','Mexico');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('64').setAttribute('title','Morocco');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('65').setAttribute('title','Mozambique');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('66').setAttribute('title','Namibia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('67').setAttribute('title','Nicaragua');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('68').setAttribute('title','Nigeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('69').setAttribute('title','Panama');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('70').setAttribute('title','Paraguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('71').setAttribute('title','Peru');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('72').setAttribute('title','Rwanda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('73').setAttribute('title','Senegal');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('74').setAttribute('title','Seychelles');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('75').setAttribute('title','Sierra Leone');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('76').setAttribute('title','Sudan');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('77').setAttribute('title','Swaziland');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('78').setAttribute('title','Tanzania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('79').setAttribute('title','Togo');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('80').setAttribute('title','Trinidad and Tobago');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('81').setAttribute('title','Tunisia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('82').setAttribute('title','Uganda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('83').setAttribute('title','United States');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('84').setAttribute('title','Uruguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('85').setAttribute('title','Venezuela');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('86').setAttribute('title','Zambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('87').setAttribute('title','Zimbabwe');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('88').setAttribute('title','Algeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('89').setAttribute('title','Angola');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('90').setAttribute('title','Argentina');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('91').setAttribute('title','Bahamas');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('92').setAttribute('title','Barbados');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('93').setAttribute('title','Belize');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('94').setAttribute('title','Benin');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('95').setAttribute('title','Bolivia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('96').setAttribute('title','Botswana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('97').setAttribute('title','Brazil');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('98').setAttribute('title','Burkina Faso');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('99').setAttribute('title','Burundi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('100').setAttribute('title','Cameroon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('101').setAttribute('title','Canada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('102').setAttribute('title','Cape Verde');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('103').setAttribute('title','Central African Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('104').setAttribute('title','Chad');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('105').setAttribute('title','Chile');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('106').setAttribute('title','Colombia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('107').setAttribute('title','Comoros');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('108').setAttribute('title','Congo, Dem. Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('109').setAttribute('title','Congo, Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('110').setAttribute('title','Costa Rica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('111').setAttribute('title','Cote d_Ivoire');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('112').setAttribute('title','Cuba');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('113').setAttribute('title','Dominican Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('114').setAttribute('title','Ecuador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('115').setAttribute('title','Egypt');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('116').setAttribute('title','El Salvador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('117').setAttribute('title','Eritrea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('118').setAttribute('title','Ethiopia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('119').setAttribute('title','Gambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('120').setAttribute('title','Ghana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('121').setAttribute('title','Guatemala');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('122').setAttribute('title','Guinea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('123').setAttribute('title','Guyana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('124').setAttribute('title','Haiti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('125').setAttribute('title','Honduras');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('126').setAttribute('title','Jamaica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('127').setAttribute('title','Kenya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('128').setAttribute('title','Lesotho');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('129').setAttribute('title','Liberia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('130').setAttribute('title','Libya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('131').setAttribute('title','Madagascar');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('132').setAttribute('title','Malawi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('133').setAttribute('title','Mali');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('134').setAttribute('title','Mauritania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('135').setAttribute('title','Mauritius');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('136').setAttribute('title','Mexico');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('137').setAttribute('title','Morocco');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('138').setAttribute('title','Mozambique');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('139').setAttribute('title','Namibia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('140').setAttribute('title','Nicaragua');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('141').setAttribute('title','Niger');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('142').setAttribute('title','Nigeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('143').setAttribute('title','Panama');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('144').setAttribute('title','Paraguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('145').setAttribute('title','Peru');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('146').setAttribute('title','Rwanda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('147').setAttribute('title','St. Lucia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('148').setAttribute('title','St. Vincent and the Grenadines');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('149').setAttribute('title','Senegal');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('150').setAttribute('title','Seychelles');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('151').setAttribute('title','Sierra Leone');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('152').setAttribute('title','Sudan');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('153').setAttribute('title','Swaziland');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('154').setAttribute('title','Tanzania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('155').setAttribute('title','Togo');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('156').setAttribute('title','Trinidad and Tobago');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('157').setAttribute('title','Tunisia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('158').setAttribute('title','Uganda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('159').setAttribute('title','United States');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('160').setAttribute('title','Uruguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('161').setAttribute('title','Venezuela');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('162').setAttribute('title','Zambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('163').setAttribute('title','Zimbabwe');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('164').setAttribute('title','Algeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('165').setAttribute('title','Angola');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('166').setAttribute('title','Argentina');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('167').setAttribute('title','Bahamas');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('168').setAttribute('title','Barbados');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('169').setAttribute('title','Belize');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('170').setAttribute('title','Benin');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('171').setAttribute('title','Bolivia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('172').setAttribute('title','Botswana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('173').setAttribute('title','Brazil');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('174').setAttribute('title','Burkina Faso');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('175').setAttribute('title','Burundi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('176').setAttribute('title','Cameroon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('177').setAttribute('title','Canada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('178').setAttribute('title','Cape Verde');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('179').setAttribute('title','Central African Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('180').setAttribute('title','Chad');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('181').setAttribute('title','Chile');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('182').setAttribute('title','Colombia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('183').setAttribute('title','Comoros');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('184').setAttribute('title','Congo, Dem. Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('185').setAttribute('title','Congo, Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('186').setAttribute('title','Costa Rica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('187').setAttribute('title','Cote d_Ivoire');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('188').setAttribute('title','Cuba');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('189').setAttribute('title','Djibouti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('190').setAttribute('title','Dominican Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('191').setAttribute('title','Ecuador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('192').setAttribute('title','Egypt');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('193').setAttribute('title','El Salvador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('194').setAttribute('title','Eritrea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('195').setAttribute('title','Ethiopia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('196').setAttribute('title','Gabon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('197').setAttribute('title','Gambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('198').setAttribute('title','Ghana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('199').setAttribute('title','Guatemala');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('200').setAttribute('title','Guinea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('201').setAttribute('title','Guinea-Bissau');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('202').setAttribute('title','Guyana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('203').setAttribute('title','Haiti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('204').setAttribute('title','Honduras');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('205').setAttribute('title','Jamaica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('206').setAttribute('title','Kenya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('207').setAttribute('title','Lesotho');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('208').setAttribute('title','Liberia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('209').setAttribute('title','Libya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('210').setAttribute('title','Madagascar');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('211').setAttribute('title','Malawi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('212').setAttribute('title','Mali');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('213').setAttribute('title','Mauritania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('214').setAttribute('title','Mauritius');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('215').setAttribute('title','Mexico');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('216').setAttribute('title','Morocco');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('217').setAttribute('title','Mozambique');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('218').setAttribute('title','Namibia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('219').setAttribute('title','Nicaragua');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('220').setAttribute('title','Niger');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('221').setAttribute('title','Nigeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('222').setAttribute('title','Panama');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('223').setAttribute('title','Paraguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('224').setAttribute('title','Peru');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('225').setAttribute('title','Rwanda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('226').setAttribute('title','St. Lucia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('227').setAttribute('title','St. Vincent and the Grenadines');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('228').setAttribute('title','Senegal');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('229').setAttribute('title','Seychelles');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('230').setAttribute('title','Sierra Leone');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('231').setAttribute('title','South Africa');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('232').setAttribute('title','Sudan');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('233').setAttribute('title','Suriname');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('234').setAttribute('title','Swaziland');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('235').setAttribute('title','Tanzania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('236').setAttribute('title','Togo');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('237').setAttribute('title','Trinidad and Tobago');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('238').setAttribute('title','Tunisia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('239').setAttribute('title','Uganda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('240').setAttribute('title','United States');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('241').setAttribute('title','Uruguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('242').setAttribute('title','Venezuela');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('243').setAttribute('title','Zambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('244').setAttribute('title','Zimbabwe');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('245').setAttribute('title','Algeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('246').setAttribute('title','Angola');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('247').setAttribute('title','Antigua and Barbuda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('248').setAttribute('title','Argentina');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('249').setAttribute('title','Bahamas');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('250').setAttribute('title','Barbados');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('251').setAttribute('title','Belize');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('252').setAttribute('title','Benin');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('253').setAttribute('title','Bolivia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('254').setAttribute('title','Botswana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('255').setAttribute('title','Brazil');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('256').setAttribute('title','Burkina Faso');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('257').setAttribute('title','Burundi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('258').setAttribute('title','Cameroon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('259').setAttribute('title','Canada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('260').setAttribute('title','Cape Verde');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('261').setAttribute('title','Central African Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('262').setAttribute('title','Chad');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('263').setAttribute('title','Chile');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('264').setAttribute('title','Colombia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('265').setAttribute('title','Comoros');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('266').setAttribute('title','Congo, Dem. Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('267').setAttribute('title','Congo, Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('268').setAttribute('title','Costa Rica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('269').setAttribute('title','Cote d_Ivoire');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('270').setAttribute('title','Cuba');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('271').setAttribute('title','Djibouti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('272').setAttribute('title','Dominican Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('273').setAttribute('title','Ecuador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('274').setAttribute('title','Egypt');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('275').setAttribute('title','El Salvador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('276').setAttribute('title','Equatorial Guinea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('277').setAttribute('title','Eritrea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('278').setAttribute('title','Ethiopia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('279').setAttribute('title','Gabon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('280').setAttribute('title','Gambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('281').setAttribute('title','Ghana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('282').setAttribute('title','Grenada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('283').setAttribute('title','Guatemala');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('284').setAttribute('title','Guinea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('285').setAttribute('title','Guinea-Bissau');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('286').setAttribute('title','Guyana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('287').setAttribute('title','Haiti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('288').setAttribute('title','Honduras');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('289').setAttribute('title','Jamaica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('290').setAttribute('title','Kenya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('291').setAttribute('title','Lesotho');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('292').setAttribute('title','Liberia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('293').setAttribute('title','Libya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('294').setAttribute('title','Madagascar');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('295').setAttribute('title','Malawi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('296').setAttribute('title','Mali');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('297').setAttribute('title','Mauritania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('298').setAttribute('title','Mauritius');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('299').setAttribute('title','Mexico');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('300').setAttribute('title','Morocco');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('301').setAttribute('title','Mozambique');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('302').setAttribute('title','Namibia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('303').setAttribute('title','Nicaragua');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('304').setAttribute('title','Niger');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('305').setAttribute('title','Nigeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('306').setAttribute('title','Panama');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('307').setAttribute('title','Paraguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('308').setAttribute('title','Peru');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('309').setAttribute('title','Rwanda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('310').setAttribute('title','St. Lucia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('311').setAttribute('title','St. Vincent and the Grenadines');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('312').setAttribute('title','Senegal');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('313').setAttribute('title','Seychelles');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('314').setAttribute('title','Sierra Leone');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('315').setAttribute('title','South Africa');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('316').setAttribute('title','Sudan');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('317').setAttribute('title','Suriname');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('318').setAttribute('title','Swaziland');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('319').setAttribute('title','Tanzania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('320').setAttribute('title','Togo');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('321').setAttribute('title','Trinidad and Tobago');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('322').setAttribute('title','Tunisia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('323').setAttribute('title','Uganda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('324').setAttribute('title','United States');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('325').setAttribute('title','Uruguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('326').setAttribute('title','Venezuela');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('327').setAttribute('title','Zambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('328').setAttribute('title','Zimbabwe');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('329').setAttribute('title','Algeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('330').setAttribute('title','Angola');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('331').setAttribute('title','Antigua and Barbuda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('332').setAttribute('title','Argentina');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('333').setAttribute('title','Bahamas');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('334').setAttribute('title','Barbados');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('335').setAttribute('title','Belize');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('336').setAttribute('title','Benin');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('337').setAttribute('title','Bolivia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('338').setAttribute('title','Botswana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('339').setAttribute('title','Brazil');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('340').setAttribute('title','Burkina Faso');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('341').setAttribute('title','Burundi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('342').setAttribute('title','Cameroon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('343').setAttribute('title','Canada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('344').setAttribute('title','Cape Verde');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('345').setAttribute('title','Central African Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('346').setAttribute('title','Chad');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('347').setAttribute('title','Chile');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('348').setAttribute('title','Colombia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('349').setAttribute('title','Comoros');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('350').setAttribute('title','Congo, Dem. Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('351').setAttribute('title','Congo, Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('352').setAttribute('title','Costa Rica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('353').setAttribute('title','Cote d_Ivoire');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('354').setAttribute('title','Cuba');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('355').setAttribute('title','Djibouti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('356').setAttribute('title','Dominican Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('357').setAttribute('title','Ecuador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('358').setAttribute('title','Egypt');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('359').setAttribute('title','El Salvador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('360').setAttribute('title','Equatorial Guinea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('361').setAttribute('title','Eritrea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('362').setAttribute('title','Ethiopia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('363').setAttribute('title','Gabon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('364').setAttribute('title','Gambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('365').setAttribute('title','Ghana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('366').setAttribute('title','Grenada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('367').setAttribute('title','Guatemala');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('368').setAttribute('title','Guinea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('369').setAttribute('title','Guinea-Bissau');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('370').setAttribute('title','Guyana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('371').setAttribute('title','Haiti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('372').setAttribute('title','Honduras');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('373').setAttribute('title','Jamaica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('374').setAttribute('title','Kenya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('375').setAttribute('title','Lesotho');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('376').setAttribute('title','Liberia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('377').setAttribute('title','Libya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('378').setAttribute('title','Madagascar');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('379').setAttribute('title','Malawi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('380').setAttribute('title','Mali');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('381').setAttribute('title','Mauritania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('382').setAttribute('title','Mauritius');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('383').setAttribute('title','Mexico');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('384').setAttribute('title','Morocco');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('385').setAttribute('title','Mozambique');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('386').setAttribute('title','Namibia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('387').setAttribute('title','Nicaragua');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('388').setAttribute('title','Niger');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('389').setAttribute('title','Nigeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('390').setAttribute('title','Panama');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('391').setAttribute('title','Paraguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('392').setAttribute('title','Peru');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('393').setAttribute('title','Rwanda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('394').setAttribute('title','St. Lucia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('395').setAttribute('title','St. Vincent and the Grenadines');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('396').setAttribute('title','Senegal');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('397').setAttribute('title','Seychelles');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('398').setAttribute('title','Sierra Leone');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('399').setAttribute('title','South Africa');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('400').setAttribute('title','Sudan');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('401').setAttribute('title','Suriname');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('402').setAttribute('title','Swaziland');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('403').setAttribute('title','Tanzania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('404').setAttribute('title','Togo');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('405').setAttribute('title','Trinidad and Tobago');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('406').setAttribute('title','Tunisia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('407').setAttribute('title','Uganda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('408').setAttribute('title','United States');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('409').setAttribute('title','Uruguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('410').setAttribute('title','Venezuela');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('411').setAttribute('title','Zambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('412').setAttribute('title','Zimbabwe');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('413').setAttribute('title','Algeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('414').setAttribute('title','Angola');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('415').setAttribute('title','Antigua and Barbuda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('416').setAttribute('title','Argentina');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('417').setAttribute('title','Bahamas');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('418').setAttribute('title','Barbados');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('419').setAttribute('title','Belize');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('420').setAttribute('title','Benin');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('421').setAttribute('title','Bolivia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('422').setAttribute('title','Botswana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('423').setAttribute('title','Brazil');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('424').setAttribute('title','Burkina Faso');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('425').setAttribute('title','Burundi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('426').setAttribute('title','Cameroon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('427').setAttribute('title','Canada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('428').setAttribute('title','Cape Verde');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('429').setAttribute('title','Central African Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('430').setAttribute('title','Chad');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('431').setAttribute('title','Chile');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('432').setAttribute('title','Colombia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('433').setAttribute('title','Comoros');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('434').setAttribute('title','Congo, Dem. Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('435').setAttribute('title','Congo, Rep.');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('436').setAttribute('title','Costa Rica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('437').setAttribute('title','Cote d_Ivoire');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('438').setAttribute('title','Cuba');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('439').setAttribute('title','Djibouti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('440').setAttribute('title','Dominican Republic');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('441').setAttribute('title','Ecuador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('442').setAttribute('title','Egypt');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('443').setAttribute('title','El Salvador');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('444').setAttribute('title','Equatorial Guinea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('445').setAttribute('title','Eritrea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('446').setAttribute('title','Ethiopia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('447').setAttribute('title','Gabon');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('448').setAttribute('title','Gambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('449').setAttribute('title','Ghana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('450').setAttribute('title','Grenada');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('451').setAttribute('title','Guatemala');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('452').setAttribute('title','Guinea');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('453').setAttribute('title','Guinea-Bissau');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('454').setAttribute('title','Guyana');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('455').setAttribute('title','Haiti');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('456').setAttribute('title','Honduras');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('457').setAttribute('title','Jamaica');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('458').setAttribute('title','Kenya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('459').setAttribute('title','Lesotho');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('460').setAttribute('title','Liberia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('461').setAttribute('title','Libya');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('462').setAttribute('title','Madagascar');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('463').setAttribute('title','Malawi');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('464').setAttribute('title','Mali');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('465').setAttribute('title','Mauritania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('466').setAttribute('title','Mauritius');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('467').setAttribute('title','Mexico');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('468').setAttribute('title','Morocco');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('469').setAttribute('title','Mozambique');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('470').setAttribute('title','Namibia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('471').setAttribute('title','Nicaragua');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('472').setAttribute('title','Niger');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('473').setAttribute('title','Nigeria');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('474').setAttribute('title','Panama');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('475').setAttribute('title','Paraguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('476').setAttribute('title','Peru');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('477').setAttribute('title','Rwanda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('478').setAttribute('title','St. Lucia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('479').setAttribute('title','St. Vincent and the Grenadines');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('480').setAttribute('title','Senegal');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('481').setAttribute('title','Seychelles');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('482').setAttribute('title','Sierra Leone');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('483').setAttribute('title','South Africa');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('484').setAttribute('title','Sudan');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('485').setAttribute('title','Suriname');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('486').setAttribute('title','Swaziland');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('487').setAttribute('title','Tanzania');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('488').setAttribute('title','Togo');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('489').setAttribute('title','Trinidad and Tobago');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('490').setAttribute('title','Tunisia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('491').setAttribute('title','Uganda');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('492').setAttribute('title','United States');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('493').setAttribute('title','Uruguay');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('494').setAttribute('title','Venezuela');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('495').setAttribute('title','Zambia');document.querySelectorAll('#svg_2708bd959ee20190912102831')[0].getElementById('496').setAttribute('title','Zimbabwe');};","uid":"svg_2708bd959ee20190912102831","ratio":1.2,"settings":{"tooltip":{"css":"{position:absolute;pointer-events:none;z-index:999;padding:5px;background:black;color:white;border-radius:2px 2px 2px 2px;}","offx":10,"offy":0,"use_cursor_pos":true,"opacity":0.9,"usefill":false,"usestroke":false,"delay":{"over":200,"out":500}},"hover":{"css":"{fill:orange;stroke:gray;}"},"zoom":{"min":1,"max":1},"capture":{"css":"{fill:red;stroke:gray;}","type":"multiple","only_shiny":true},"toolbar":{"position":"top","saveaspng":false},"sizing":{"rescale":true,"width":0.75}}},"evals":[],"jsHooks":[]}</script>

<!--/html_preserve-->

# A more complicated example (for showing the capabilities of ggplot2)

``` r
west <- c("Western Europe","Northern Europe","Southern Europe",
          "Northern America","Australia and New Zealand")

gapminder <- gapminder %>%
  mutate(group = case_when(
    region %in% west ~ "The West",
    region %in% c("Eastern Asia", "South-Eastern Asia") ~ "East Asia",
    region %in% c("Caribbean", "Central America", "South America") ~ "Latin America",
    continent == "Africa" & region != "Northern Africa" ~ "Sub-Saharan Africa",
    TRUE ~ "Others"))
gapminder <- gapminder %>%
  mutate(group = factor(group, levels = rev(c("Others", "Latin America", "East Asia","Sub-Saharan Africa", "The West"))))

filter(gapminder, year%in%c(1962, 2013) & !is.na(group) &
         !is.na(fertility) & !is.na(life_expectancy)) %>%
  mutate(population_in_millions = population/10^6) %>%
  ggplot( aes(fertility, y=life_expectancy, col = group, size = population_in_millions)) +
  geom_point(alpha = 0.8) +
  guides(size=FALSE) +
  theme(plot.title = element_blank(), legend.title = element_blank()) +
  coord_cartesian(ylim = c(30, 85)) +
  xlab("Fertility rate (births per woman)") +
  ylab("Life Expectancy") +
  geom_text(aes(x=7, y=82, label=year), cex=12, color="grey") +
  facet_grid(. ~ year) +
  theme(strip.background = element_blank(),
        strip.text.x = element_blank(),
        strip.text.y = element_blank(),
   legend.position = "top")
```

![](demo_files/figure-gfm/unnamed-chunk-31-1.png)<!-- -->

# Data Distributions & Outliers

## Detecting outliers

For this part we use a different and more simple dataset This dataset
contains 1192 observations on self-reported:

  - `height` (inch)
  - `earn` ($)
  - `sex` (gender)
  - `ed` (currently unannotated)
  - `age` (years)
  - `race`

<!-- end list -->

``` r
heights_data <- read_csv(file = here::here("data",
                                          "heights_outliers.csv"))

heights_data
```

    ## # A tibble: 1,192 x 6
    ##     earn height sex       ed   age race    
    ##    <dbl>  <dbl> <chr>  <dbl> <dbl> <chr>   
    ##  1 50000   74.4 male      16    45 white   
    ##  2 60000   65.5 female    16    58 white   
    ##  3 30000   63.6 female    16    29 white   
    ##  4 50000   63.1 female    16    91 other   
    ##  5 51000   63.4 female    17    39 white   
    ##  6  9000   64.4 female    15    26 white   
    ##  7 29000   61.7 female    12    49 white   
    ##  8 32000   72.7 male      17    46 white   
    ##  9  2000   72.0 male      15    21 hispanic
    ## 10 27000   72.2 male      12    26 white   
    ## # ... with 1,182 more rows

## Data characteristics

We will focus on the variable `height` here

``` r
summary_heights_data <- heights_data %>%
  group_by(sex, age) %>%
  summarise(mean_height = mean(height, na.rm = TRUE),
            min_height = min(height),
            max_height = max(height)) %>%
  arrange(desc(mean_height))

summary_heights_data[c(1:4),]
```

    ## # A tibble: 4 x 5
    ## # Groups:   sex [2]
    ##   sex      age mean_height min_height max_height
    ##   <chr>  <dbl>       <dbl>      <dbl>      <dbl>
    ## 1 female    55       141.        61.9      664. 
    ## 2 male      39       134.        66.6      572. 
    ## 3 male      55        73.2       71.7       74.8
    ## 4 male      91        73.1       73.1       73.1

From the above summary we can conclude that there are two outliers
(presumably entry errors).

**Calculate the height in meters for each outlier in the `Console`** 1
inch = 0,0254 meters

**Please discuss the solution with your neighbour**

## Checking the frequency distribution

``` r
heights_data %>%
  ggplot(aes(x = height)) +
  geom_histogram(aes(stat = "identity"), bins = 200)
```

![](demo_files/figure-gfm/unnamed-chunk-34-1.png)<!-- -->

This distribution looks odd. When you see a large x-axis with no data
plotted on it, it usually means there is an outlier. If you look
carefully, you will spot two outliers around 600

## Boxplots to detect outliers

``` r
heights_data %>%
  ggplot(aes(y = height)) +
  geom_boxplot()
```

![](demo_files/figure-gfm/unnamed-chunk-35-1.png)<!-- -->

So apparantly there is one data point that is way off from the rest of
the distribution. Let’s remove this point, using `filter()` from the
`{dplyr}` package like we did before on the `gapminder` dataset.

``` r
heights_data %>%
  dplyr::filter(height < 100) %>%
  ggplot(aes(y = height)) +
  geom_boxplot()
```

![](demo_files/figure-gfm/unnamed-chunk-36-1.png)<!-- -->

``` r
## by sex

heights_data %>%
  dplyr::filter(height < 100) %>%
  ggplot(aes(y = height, x = sex)) +
  geom_boxplot()
```

![](demo_files/figure-gfm/unnamed-chunk-36-2.png)<!-- -->

## New frequency distribution

Now let’s plot a new distribution plot, this time we plot density,
leaving the outlier out

``` r
heights_data %>%
  dplyr::filter(height < 100) %>%
  ggplot(aes(height)) +
  geom_freqpoly(aes(y = ..density..))
```

![](demo_files/figure-gfm/unnamed-chunk-37-1.png)<!-- -->

``` r
## by sex
heights_data %>%
  dplyr::filter(height < 100) %>%
  ggplot(aes(height)) +
  geom_freqpoly(aes(y = ..density.., colour = sex))
```

![](demo_files/figure-gfm/unnamed-chunk-37-2.png)<!-- -->

## A formal check to detect an outlier - handle with care

The best way to spot and take care of an outlier (**you never delete an
outlier unsubstantiated\!\!**) is to create multiple visualizations as
decribed above. However, there is a formal, simple statistical apporach
to assess whether an observation is an outlier. The apporach below can
only be used to detect a single outlier in a numeric vector. For more
complex (multivariate) situations see e.g.:
<http://r-statistics.co/Outlier-Treatment-With-R.html>

### The Dixon’s Q Test for detecting outliers

The `{outliers}` package contains a number of tools. Here we will
demonstrate the `outlier()` and `dixon.test()` functions. For more
information on the DQT, see:
<https://www.statisticshowto.datasciencecentral.com/dixons-q-test/>

### `outlier()` function

This function finds the value with largest difference between it and
sample mean, which can be an outlier.

We will walk trough a simple exercise that first shows the result of the
`outlier()` function and than we calculate it by hand

First we create dummy data for a triplicate series of measurements. We
assume 5 measurement values, replicated three times, containing one
outlier. We also assume is was a concentration depended series in which
the index indicates the used concentration.

``` r
set.seed(123)
## create data
random_numbers <- rnorm(n = 15, mean = 2.5, sd = 0.9) 
## inject outlier
random_numbers[4] <- 9.54
## define triplicates
triplicates <- rep(1:3, times = 5)
index <- rep(1:5, times = 3)

## put in dataframe
measured_data <- tibble(measured = random_numbers, replicate = triplicates,
                        index = index)
measured_data
```

    ## # A tibble: 15 x 3
    ##    measured replicate index
    ##       <dbl>     <int> <int>
    ##  1     2.00         1     1
    ##  2     2.29         2     2
    ##  3     3.90         3     3
    ##  4     9.54         1     4
    ##  5     2.62         2     5
    ##  6     4.04         3     1
    ##  7     2.91         1     2
    ##  8     1.36         2     3
    ##  9     1.88         3     4
    ## 10     2.10         1     5
    ## 11     3.60         2     1
    ## 12     2.82         3     2
    ## 13     2.86         1     3
    ## 14     2.60         2     4
    ## 15     2.00         3     5

``` r
## sort data
measured_data %>% 
  arrange(desc(measured))
```

    ## # A tibble: 15 x 3
    ##    measured replicate index
    ##       <dbl>     <int> <int>
    ##  1     9.54         1     4
    ##  2     4.04         3     1
    ##  3     3.90         3     3
    ##  4     3.60         2     1
    ##  5     2.91         1     2
    ##  6     2.86         1     3
    ##  7     2.82         3     2
    ##  8     2.62         2     5
    ##  9     2.60         2     4
    ## 10     2.29         2     2
    ## 11     2.10         1     5
    ## 12     2.00         3     5
    ## 13     2.00         1     1
    ## 14     1.88         3     4
    ## 15     1.36         2     3

## Plotting the data reveals the outlier.

``` r
## plot
measured_data %>% 
#  enframe() %>%
  ggplot(aes(x = index, y = measured)) +
  geom_point() +
  geom_point(data = dplyr::filter(.data = measured_data, measured > 5), 
             colour = "red", size = 6, shape = 4)
```

![](demo_files/figure-gfm/unnamed-chunk-39-1.png)<!-- -->

## Identify with `outliers::outlier()`

``` r
outliers::outlier(measured_data$measured)
```

    ## [1] 9.54

### Identify with `outliers::dixon.test()`

``` r
## note that these functions from the {outliers} package take a vector (use $)
outliers::dixon.test(measured_data$measured)
```

    ## 
    ##  Dixon test for outliers
    ## 
    ## data:  measured_data$measured
    ## Q = 0.7472, p-value < 2.2e-16
    ## alternative hypothesis: highest value 9.54 is an outlier

The test is significant against the NULL hypothesis that the highest
value is not an outlier. We can conclude that this NULL hypothesis does
not hold on the basis of the significant p-value \< 0.05

### Manual calculations:

*`outlier()`*

``` r
mean_vctr <- mean(measured_data$measured)

distance <- measured_data$measured - mean_vctr

ind <- distance == max(distance)

measured_data[ind,]
```

    ## # A tibble: 1 x 3
    ##   measured replicate index
    ##      <dbl>     <int> <int>
    ## 1     9.54         1     4

*`dixon.test()`*

The Dixon’s Q Test can be calculated manually following the following
procedure:

How to Run Dixon’s Q Test (R10).

Note: make sure your data set is normally distributed before running the
test; for example, run a Shapiro-Wilk test. Running it on different
distributions will lead to erroneous results. An extreme value may throw
off any test for normality, so try running that test without the suspect
data item. If your data set still doesn’t meet the assumption of
normality after running a test for it, then you should not run Dixon’s Q
Test.

Note that for very small sample sizes the assumtion for Normal
distribution usually does not hold. We will see more formal way of
establishing Normal distribution below.

**Caution: the test should not be used more than once for the same set
of data.**

To perform the calculations:

Step 1: Sort your data into ascending order (smallest to largest).

Step 2 :Find the Q statistic using the following formula:

\(Qexp = \frac{x2 - x1}{xn - x1}\)

Where

  - \(x1\) is the smallest (suspect) value
  - \(x2\) is the second smallest value
  - \(xn\) is the largest value

Dixon’s Q Test: Definition, Step by Step Examples + Q Critical Values
Tables

Find Outliers \> Dixon’s Q Test What is Dixon’s Q Test?

Dixon’s Q test, or just the “Q Test” is a way to find outliers in very
small, normally distributed, data sets. Small data sets are usually
defined as somewhere between 3 and 7 items. It’s commonly used in
chemistry, where data sets sometimes include one suspect observation
that’s much lower or much higher than the other values. Keeping an
outlier in data affects calculations like the mean and standard
deviation, so true outliers should be removed.

Dixon came up with many different equations to find true outliers. The
most commonly used one is called the R10 or simply the “Q” version,
which is used to test if one single value is an outlier in a sample size
of between 3 and 7. Dean and Dixon did suggest various other formulas in
a later paper, but these are not commonly used. For a full list of
alternate formulas for different sample sizes (up to about 30), go to:
Dixon’s Test, Alternate Formulas and Tables. How to Run Dixon’s Q Test
(R10).

Note: make sure your data set is normally distributed before running the
test; for example, run a Shapiro-Wilk test. Running it on different
distributions will lead to erroneous results. An extreme value may throw
off any test for normality, so try running that test without the suspect
data item. If your data set still doesn’t meet the assumption of
normality after running a test for it, then you should not run Dixon’s Q
Test,

Caution: the test should not be used more than once for the same set of
data.

Step 1: Sort your data into ascending order (smallest to largest). 167,
177, 180, 181, 185, 188, 189.

Step 2 :Find the Q statistic using the following formula: dixon’s q test
statistic

Where:

    x1 is the smallest (suspect) value,
    x2 is the second smallest value,
    and xn is the largest value.

Step 3: Find the Q critical value in the Q table (see below). For a
sample size of 3 and an alpha level of 5%, the critical value is 0.970.

Step 4: Compare the Q statistic from Step 2 with the Q critical value in
Step 3. If the Q statistic is greater than the Q critical value, the
point is an outlier.

Let’s try it on our `measured_data`

``` r
measured_data_ranked <- measured_data %>%
  arrange(measured) 
measured_data_ranked$measured
```

    ##  [1] 1.361445 1.881832 1.995572 1.999743 2.098904 2.292840 2.599614
    ##  [8] 2.616359 2.823832 2.860694 2.914825 3.601674 3.902837 4.043558
    ## [15] 9.540000

``` r
x1 <- measured_data_ranked$measured[1]
x2 <- measured_data_ranked$measured[2]
xn <- max(measured_data_ranked$measured)

Qexp = (x2-x1) / (xn-x1)

##
0.5/8
```

    ## [1] 0.0625

## Checking normality with a qqplot

a qqplot provides a visual aid to assess whether a distribution is
approaching normality

``` r
## 
source(file = here::here("code", "ggqq.R"))
height_data_outlier_removed <- heights_data %>%
  dplyr::filter(height < 100)
  gg_qq(height_data_outlier_removed$height) 
```

![](demo_files/figure-gfm/unnamed-chunk-44-1.png)<!-- -->

    ##       25%       75% 
    ## 66.926998  4.328462

``` r
## formal statistical proof
shapiro.test(height_data_outlier_removed$height)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  height_data_outlier_removed$height
    ## W = 0.98485, p-value = 8.491e-10

all data -\> reject hypothesis that the sample has a normal distribution

## Test individual distributions

``` r
males <- height_data_outlier_removed %>%
  dplyr::filter(sex == "male")

females <- height_data_outlier_removed %>%
  dplyr::filter(sex == "female")

shapiro.test(males$height)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  males$height
    ## W = 0.99053, p-value = 0.002532

``` r
shapiro.test(females$height)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  females$height
    ## W = 0.99277, p-value = 0.002105

``` r
## add shapiro for each sex

## we can do the same for age
shapiro.test(males$age)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  males$age
    ## W = 0.93358, p-value = 3.506e-14

``` r
shapiro.test(females$age)
```

    ## 
    ##  Shapiro-Wilk normality test
    ## 
    ## data:  females$age
    ## W = 0.93978, p-value = 4.862e-16
