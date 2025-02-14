---
title: >-
  Data Cleaning with R: Navigating Challenges and Answering Key Questions
date: 2025-02-01
image:
  focal_point: 'top'
  shape: rounded
editor_options: 
  markdown: 
    wrap: 100
---



<!--more-->

As a PhD student, I have used R for a handful of projects and become comfortable with it.
I chose to use R instead of SPSS to analyze data for my thesis. 
For my thesis, I used data collected for the Midlife in the United States (MIDUS) study— a longitudinal examination of health, well-being,
and social responsibility among middle-aged adults in the United States. 
My research question involved exploring stress, race, and cardiac vagal tone (CVT) /heart rate variability (HRV). 
MIDUS is a large project that contains multiple waves and subprojects.

Because I was working with data collected at two different time points for four different samples, 
the data I downloaded contained over 9000 observations with almost 400 variables. 
All observations weren’t relevant to my question and many variables were duplicates. 
Most of the challenges I had involved figuring out the most efficient way to combine 
information into a single variable. Between several Google searches (stackoverflow was helpful) and ChatGPT, 
I was able to find a method that worked. Hopefully, if these are questions you have, it will take you less time. 
You can find the full example on my Github.

**How do you create a single variable from more than two variables? For example, if you have four
variables that contain information about sex, how do you combine them into one variable?**

I knew that I could use ifelse() or if_else() for two variables but was unsure of how to do this with more than two variables. I learned about case_when () from a data visualization course and decided to try it. For the sample code below, I wanted the value that was present to be used for the new variable. Since the information was from different waves/participants, if there was a value present for one variable, it would not be present for any of the other variables. With the case_when() function, I set up the condition that if there was a value greater than 0, that value would be used for the new variable. 

```         
# Use when there are only two variables
stress = ifelse (is.na (B4QPS_PS), RA4QPS_PS,B4QPS_PS)

# Use when there are more than two variables. For this variable, NA was used if it
# did not apply to the case so I set the condition to be greater than 0 because if 
# there was a value present for a specific variable, then there wouldn't
# be a value for another variable. 

 educ = case_when(B1PB1 > 0 ~ B1PB1, # M2P1 education
                     BACB1 > 0 ~ BACB1, # Milwaukee 1 education
                     RA1PB1 > 0 ~ RA1PB1, # MR 1 education
                     RAACB1 > 0 ~ RAACB1), # Milwaukee R education
```

Also, I used case_when () for creating a new variable from multiple variables that needed 
to meet different conditions simultaneously. For example, I wanted to create a tobacco user 
variable that categorized participants as current users, previously users, or never used. 
The goal was to accurately capture tobacco usage and have only respondents who had no response 
for any of the smoking/tobacco variables be marked as NA. This was a bit challenging because the 
information was spread across four variables, and I had to figure out the best way to set up the 
code so that it accurately distinguished between non-users, users, and previous users. 
Unfortunately, I wasn’t able to find a solution using Google or ChatGPT and decided to use case_when ().

```         
# I started with the easiest logic formulas, which were coding the current user and non-user. 
# Then, I spent some time figuring out how to code the quitters. After that, there were still  cases
# that weren't categorized. After some trial-and-error, I figure out how to set up the code to categorize them correctly.
  
  mutate (tobacco_user = case_when(evertobacco == 2 & eversmoke == 2 ~ "nonuser", # NO coded as 2. 
                                  currtobacco == 1 | currsmoke == 1 ~ "user",  # YES coded as 1.
                                  eversmoke == 1 & currsmoke == 2 ~ "quit",
                                  evertobacco == 1 & currtobacco == 2 ~ "quit",
                                  is.na (evertobacco) & eversmoke== 2 & 
                                    currtobacco == 2 ~ "nonuser",
                                  evertobacco == 2 & is.na (currtobacco) & 
                                    is.na(eversmoke) & is.na (currsmoke) ~"nonuser")
```

**How do you create a single variable from 15 variables such that all the variables start with the
same thing followed by some sequence?**

The MIDUS team conducted a comprehensive medication evaluation resulting in 15 medication 
variables for each participant. The first thing I wanted to do was rename the variables 
so that they were named meds1-meds15. I knew how to use paste0 () but was unsure of the 
most efficient way to accomplish this task. Initially, I created the variable I needed and 
conducted the analysis without renaming the variables. After completing the analysis, I decided 
to try to rename the variables and I asked ChatGPT. ChatGPT suggested that I use rename_with () 
and seq_along (), which did exactly what I wanted.

```         
rename_with (., ~ paste0("med", seq_along(.)), starts_with('B4XPICD9M')) 
```

**How do I replace different values that represent NA with NA?**

Different values were used to represent NA, which made converting these values to NA challenging. 
For example, for some variables 7 or 8 may have been used while for other variables 999 or 9998 may have been used. 
I wanted to convert these to NA so that they would not be calculated when I ran analyses. 
However, I was unsure how to do this without unintentionally removing valid numbers. 
I did a Google search and the solution I found was to create objects and then use mutate (across ()).

```         
# create objects for values ---------------------------------------------------------------

missing_values_high <- 
  c(97, 98,99)

missing_values_low <- 
  c(7,8,9)

# create objects for variables -----------------------------------------------------------

missing_variables_high <- c( "educ", "resprate", "stress", 'logRMSSD1','logRMSSD2', "daily5drinks")

missing_variables_low <- 
  c("heartdisease", "highBP", "heartmurmur", "tiastroke", "cholesterol", 
                "diabetes", "alcoholism", "depression", "sleepdur", "sleepqual",
                "race1", "race2", "currsmoke", "currtobacco",  "eversmoke" , 
                "evertobacco", "drinkpastmonth", 'logRMSSD1','logRMSSD2',
    "soc_anxiety", "waisthip")

# replace missing values with NA---------------------------------------------------------

missing_converted <- 
  merged_data %>% 
  mutate(
    across( # applies mutation to multiple variables/columns
      c(missing_variables_high, MASQgendisdep : spieltraitanxinv), # can use this or all_of ()
      ~replace (., . %in% missing_values_high, NA)), # replace in dataset-variables in object - with NA
    across(
      all_of(missing_variables_low), #can use this or c ()
      ~replace(., . %in% missing_values_low, NA)) # replace in dataset-variables in object - with NA
    ) 
```

**How do I create a single variable from multiple variables that meet the same condition?**

For one question, I asked about how to create a single variable from multiple variables meeting different conditions. 
This time, I wanted to create a single variable where if any of the variables met a single condition, 
it resulted in a YES and if none of the variables did, it resulted in a NO. As I previously mentioned, 
I had 15 variables for medication and wanted to create a binary variable that indicates whether at 
least one of the variables meets a specific condition. When I did this for my thesis, 
I initially used a case_when () statement. However, after my thesis, I searched to see if there might 
be a better solution and found that apply ( ) also worked. While I found case_when () to be more straightforward, 
apply () was simpler. 

```         
# Option 1------------------------------------

bpmeds = case_when( 
  totalmeds == 0 ~ 0,
  B4XPICD9M1 == 401 | B4XPICD9M2 ==401 | 
  B4XPICD9M3 == 401 | B4XPICD9M4 ==401 | 
  B4XPICD9M5==401 | B4XPICD9M6 == 401 | 
  B4XPICD9M7 == 401 | B4XPICD9M8 == 401 | 
  B4XPICD9M9 == 401 |B4XPICD9M10 == 401 | 
  B4XPICD9M11 == 401 | B4XPICD9M12 == 401 | 
  B4XPICD9M13 == 401 | B4XPICD9M14 == 401 | 
  B4XPICD9M15 ==401 ~ 1, .default = 0)


# Option 2 ------------------------------------

meds_function <- function (x) x == 401

updated_merged_bp <- 
  updated_merged %>% 
This function allowed me to apply the same condition to multiple variables.
Reads select all variables that start "med", 1 = apply over all select rows, 
for any that meet the condition x== 401
   mutate(bp_meds = apply(select(., starts_with("med")), 1, function(x) any(meds_function(x))), 
          bp_meds_2 = case_when(bp_meds == 1 ~ 1, .default = 0)) 
```

**Conclusion**

Although I was comfortable with R prior to using it for my thesis, using R to conduct secondary data analysis 
was a great learning experience. With R, I know there are multiple ways to do the same thing, 
and I wanted to find the most concise way to code even if I knew of a method that worked. 
I know that there are probably more efficient ways to achieve what I was trying to do and welcome any feedback. 

