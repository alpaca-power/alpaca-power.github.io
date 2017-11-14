---
title:  "P-value elimination in R"
date:  2017-09-29 02:34:00
category: [r]
tags: [r]
---

_Update: Ok... I know p-value elimination for model-building is deeply flawed per https://robjhyndman.com/hyndsight/crossvalidation/, though this nonetheless helped me save a lot of time during my midterms~_


So, I'm taking the Applied Modern Statistical Learning Methods class at USC Marshall, and we have a homework where we're going to build a regression using a dataset with 38 variables. Right from the get go, we were to exclude 5 variables that were categorical/unique etc.

I hate typing things over and over again, and I'm not a fan of copy & paste. Thus, it took me some researching to figure out how to exclude variables in a list using the `dplyr` package. Really handy.

```r
# I replaced the actual variable names with var

excluded_vars <- c('var1', 'var2', 'var3', 'var4', 'var5')

df.lm <- glm(yvar ~ .,
            data = select(df.training, -one_of(excluded_vars)),
            family = 'binomial')
```

I was pleased at my progress (which really could have been much faster if I just typed `-var1 -var2 -etc` in the regression model or using `select`, but I was so set on having a list for some reason). Then I saw the model's `summary`, and was unhappy again. That's still at least 30 variables, and with a quick look there were around 10 predictor variables with p-values greater than 0.05. Frankly, looking at those numbers made my eyes hurt.

Thus, I decided to try writng code that would automatically identify the predictor variable with the greatest p-value. This, again, took me more time than simply squinting at the numbers and finding the largest p-value...but no regrets.

```r
## first extract predictor coefficient values
df.lm.summary <- as.data.frame(summary(df.lm)$coefficients)

## isolate variable with largest p-value
coefs <- df.lm.summary %>%
              mutate(variable = rownames(df.lm.summary)) %>%
              arrange(desc(`Pr(>|z|)`)) %>%
              filter(row_number() == 1)
```

Again, I thought I'd be satisfied... but then I would need to exclude these automatically identified variables over and over again through very manual typing (or copy & paste), which will result in a very long code/R markdown file. Once again I was very annoyed by the prospect of this manual work, so I naturally tried to find a way of automating this menial task with a `for` loop. After ~30 minutes of trying to solve this pesky problem, I finally figured out how to do it with `for` loops! (By the way, I do not really like R's for loops...) 

The final code I used to build that regression model is as follows:

```r
# Automating the model building based on significance level of 0.05

## initialize dataset to use
df.training.new <- select(df.training, -one_of(excluded_vars))

## initialize data frame to record largest p-value in regression model, nrow = ncol() because it represents number of variables
pvalues <- matrix(NA, nrow = ncol(df.training.new), ncol = 2, dimnames = list(NULL,c('variable', 'pvalue'))) 

pvalues <- as.data.frame(pvalues)

for(i in 1:ncol(df.training.new)){
  
  df.lm <- glm(yvar ~ .,
                   data = df.training.new,
                   family = 'binomial')
  
  ## extract variable summary
  df.lm.summary <- as.data.frame(summary(df.lm)$coefficients)
  
  ## isolate variable with largest p-value
  coefs <- df.lm.summary %>%
              mutate(variable = rownames(df.lm.summary)) %>%
              arrange(desc(`Pr(>|z|)`)) %>%
              filter(row_number() == 1)
  
  ## write variable name and p-value to dataframe
  pvalues[i,1] <- coefs$variable
  pvalues[i,2] <- coefs$`Pr(>|z|)`
  
  variable <- as.character(coefs$variable) # predictor variable with largest p-value in i iteration of regression
  pvalue <- as.numeric(coefs$`Pr(>|z|)`) # largest p-value in i iteration of regression
  
  
  ## remove the variable with largest p-value
  df.training.new <- df.training.new %>%
    select(-one_of(variable)) 
  
  ## end loop if the largest p-value is less than 0.05
  if (pvalue <= 0.05) break
  
}

## variables to exclude based on pvalues dataframe
exclude <- pvalues %>%
  filter(pvalue >= 0.05) %>% #because on variable will have p-value less than 0.05
  select(variable) %>% #only need variable names
  t() %>% # transform so it can be turned into a vector
  as.vector() 

excluded_vars.final <- c(excluded_vars, exclude) #combine vectors to form list of variables to exclude


df.lm.final <- glm(yvar ~ .,
            data = select(df.training, -one_of(excluded_vars.final)),
            family = 'binomial')

summary(df.lm.final) # all predictor variables in final summary results have p-values <= 0.05
```

Needless to say, I am very pleased with this method, even though I could've just done the same thing much faster if I manually looked at every summary table and typed/copy-pasted many iterations of the regression model. But the thought of doing such a repetitive task makes me feel lazy, so I would much rather spend time figuring out how to automate this p-value elimination hell.... Maybe there is an existing library/function that does something similar, but it felt good figuring this out.

I shall use this process in the future if I, for some reason, ever need to use regression on a dataset with many variables again~
