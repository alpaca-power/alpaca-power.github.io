---
title:  "Combining Raw DNA files from 23andMe & AncestryDNA"
date:  2018-03-04 15:00:00
category: [r]
tags: [r]
---

Recently I've been very interested in admixture/geneology, though sadly results for East Asian heritage are lacking in terms of detail
and usefulness if you test with a Western company. 
Wegene results are kinda sketchy in terms of QA in my opinion, not to mention their standards for calculating admixture are somewhat
questionable especially for imported data, but still quite interesting with a somewhat active community which provides some insight into
how different provinces have different admixture proportions.

I did a 23andme test last year during Black Friday, but soon became quite doubtful at results when I learned how little they tested on ancestry autosomal markers for v5 chips, thanks to their pivot to Caucasian-centric healthcare. I was quite unhappy at how I could not have a comparable admixture result with others due to low genotype rate.

A month ago I did AncestryDNA test just for comparison reasons before they also followed 23andme's steps in using customized GSA Chips.
Their ethnicity calc is still a joke for East Asians but I already expected that to be the case. 
Genotype rate was much higher at .88 for some calcs, and now I wanted to combine both results for some DIYDodecad admixture calculations
because...well, there were still some differences between the two results, magnitude of difference depends on the calc. Not much difference for Eurogenes K36, for example.

There are tools to convert AncestryDNA to 23andMe v3 format so I just used one of them (found on GitHub), but no tool to combine raw data,
presumably because it's kinda a niche interest and combined files don't work properly for other sites that take in raw data. But, being
someone who wants as accurate/detailed as possible results, I saw some people would go about combining the raw data using Excel Vlookup and I think managed to do it properly using R, I don't want to deal with Excel for large files.

Definitely think there are better ways to approach this problem, but since I'm not going to do this task at scale (just combining 2 files
for myself), I'll just go with what I have as below:

```r
library(dplyr)
library(magrittr)
library(stringi)

ancestry <- read.csv('ancestry23.txt', header = T, sep = '\t') # based on AncestryDNA file converted to 23andMe format
me <- read.csv('23andme.txt', header = T, sep = '\t')


### Preparing Raw Data ####

ancestry.2 <- ancestry # I did this in case I messed up and I don't want to import massive file again
ancestry.2$rsid <- sapply(ancestry.2$rsid, as.character) # convert factor to string for matching/combining data tables
ancestry.2$genotype <- sapply(ancestry.2$genotype, as.character) 
ancestry.2 %<>%
  mutate(genotype = if_else(genotype == '00', 'NA', genotype)) # assuming 00 is no call

ancestry.2[ancestry.2 == 'NA'] = NA # need to convert back to true NA
sum(is.na(ancestry.2)) # just checking if things work


me.2 <- me # same process as AncestryDNA preparation
me.2$rsid <- sapply(me.2$rsid, as.character)
me.2$genotype <- sapply(me.2$genotype, as.character)
me.2$chromosome <- sapply(me.2$chromosome, as.integer)

me.2 %<>%
  mutate(genotype = if_else(genotype == '--', 'NA', genotype))

me.2[me.2 == 'NA'] = NA

sum(is.na(me.2))

#### Combining Raw Data ####

new <- rbind(select(me, rsid), select(ancestry, rsid))
new <- unique(new) # the table of rsids to match to

 # matching 23andme valid calls to the master table
new.1 <- merge(new, filter(me.2,!is.na(genotype)), 
               by = 'rsid', all.x = T)
               
new.1b <- filter(new.1, !is.na(genotype)) # isolated valid calls to combine 

# new table for matching AncestryDNA results, isolated rsids with no calls.
# I prioritized 23andme results as I assumed they are more trustworthy with their QA/QC
new.2 <- new.1 %>%
  filter(is.na(genotype)) %>%
  select(rsid) 

# Matching AncestryDNA results to remaining rsids
new.2b <- merge(new.2, ancestry.2, by = 'rsid', all.x = T)

#combining both raw dna results
new.3 <- rbind(new.1b, new.2b)

# converted NA back to 23andme no-call format
new.3b[is.na(new.3b)] = '--'

# checking if things are working
sum(is.na(new.3b))

# output to new file, I had to find/replace quotes using sublime text
write.table(new.3b, 'combined.txt', sep = '\t', row.names = F)


#### Checking any Difference between Raw Data ####

# creating table of rsids both companies tested on
same <- merge(select(filter(me.2, !is.na(genotype)), rsid), select(filter(ancestry.2, !is.na(genotype)), rsid))

same.me <- merge(same, select(me.2,rsid,genotype), by = 'rsid')  # matched 23andme values to table
same.all <- merge(same.me, select(ancestry.2, rsid, genotype), by = 'rsid') # matched Ancestry results to table

diff <- same.all %>% 
  filter(genotype.x != genotype.y) %>% # first isolate the different results
  mutate(genotype.y = stri_reverse(genotype.y)) %>%  # AncestryDNA has some flipped SNPs that are same as 23andme so I flipped them back
  filter(genotype.x != genotype.y) # Final difference between two results

nrow(diff)/nrow(same) * 100 # check % of conflicting results, out of rsids both companies tested, .074% of results were different for mine

names(diff) <- c('rsid', '23andMe', 'AncestryDNA')  # since I want to save this for future reference I changed column names
write.csv(diff, 'difference.csv')
```

The actual result of my combination was...well, interesting on a minor level. Of course the numbers aren't going to differ hugely as 
that would be a problem, though in my mind 3~4% difference is still quite different especially when comparing with people 
with *very* similar admix structures. 

Thus, I got more crisp-clear results (in my mind), but overall I don't think testing with 2 companies is worth it for most people...


