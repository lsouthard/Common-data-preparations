# Common-data-preparations
_Even data needs a SPA day_

In my work, I have to do a lot of data cleaning. Here are some problems with solutions that I have come across:

## 1. Releveling + Changing Class

This may seem really simple, but the ```fct_relevel``` function is something I've adopted as a standard before I go to model data. I try to use it in conjunction with other data prep to take down runtime. I have a few examples below. I like to do this before modeling so I know exactly which level of my variable will become my reference group later on. 

Here is one simple example:
```
data %>% 
  mutate(sex = fct_relevel(as.factor(sex), c("Male", "Female", "Unknown")))
```

## 2. Regrouping, collapsing, binning Data
I often have to use variables with many levels and condense a few groups into an "other" category or I need to regroup them entirely. 

**Less than 3 Options**

You can use the ```if_else()``` if you have 2 options
* a good example of this is biological sex:
```
data %>% 
  mutate(gender = if_else(gender == "F", "Female", "Male")
```

You have 3+ options you can use ```case_when()```
* a good example of this may be gender identity

```
data %>% 
  mutate(gender = case_when(gender == "F" ~ "Female",
                            gender == "M" ~ "Male", 
                            gender ==  "NB" ~ "Non-Binary",
                            T ~ gender) #this will print whatever else is not defined.
```

**More than 3 Options**
I don't like to type that much so when my categories exceed 3ish options, I move to use ```case_when()``` with ```%in%``` if they are clean and ```grepl()``` if they aren't clean. 

```
data %>% 
  mutate(gender = fct_relevel(case_when(gender %in% c("Female", "Male")~ as.character(gender), T ~ "Other")))
```
I use ```fct_relevel()``` because I'm usually preparing for analysis. This also let's me later use the ```as.numeric(as.factor(gender))``` transformation on my data if I need to conver the class and I will know exactly what my levels are. 

This is great and everything, but often my data isn't in nice categories because a lot of our data sources call for write-in data. In this case, I use ```grepl()``` from ```base()```. Here's an example when students were asked to write in their high school. This data originally had ~100 different _real_ observations, but with spelling errors it came out to much more than that:

```
data %>% 
  mutate(hs.type = fct_relevel(case_when(HIGH_SCHOOL == "GED" ~ "GED",
                                         grepl("Adult", HIGH_SCHOOL) ~ "Adult",
                                         HIGH_SCHOOL %in% c("Veterans Tribute Career/Tech", "Utah School For Deaf And Blind"
                                                            ,"Alpine Summit Programs", "Horizonte Instr Trng Ctr"
                                                            ,"Ombudsman Educational Services"
                                                            ,"Opportunities For Learning "
                                                            ,"Life Skills Center Co Springs"
                                                            ,"Career Success Schools"
                                         ) ~ "Alternative",
                                         grepl("Charter|Saint|Academy|Acad", HIGH_SCHOOL) ~"Prepped",
                                         grepl("Online|Home|Independent", HIGH_SCHOOL) ~"Home",
                                         HIGH_SCHOOL == "Out of Country HS" ~ "Non-US",
                                         T ~ "Traditional"),
                               "Traditional"))
```
I leveraged the ```%in%``` operator + ```grepl()``` to re-group the different high school types. I had left the alternative group to show the how you can define a list as well. 
