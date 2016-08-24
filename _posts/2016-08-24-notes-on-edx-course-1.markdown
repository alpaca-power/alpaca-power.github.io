---
title:  "Notes on Module2 of Python for Data Science edX Course"
date:  2016-08-24 00:32:00
category: [python]
tags: [python]
---

Ok...I got stuck on the Lab 3 questions for Module 2 of the "DAT210x Programming with Python for Data Science" course. In particular, I got stuck at Q2 around July 15 and didn't proceed forward since I was going to LA on July 19 to obtain my MBA degree at USC. 

_(Just to remind myself, Q1's solution was:_

```python
df.read_csv('filepath', header=None, names=['motor', 'screw', 'pgain', 'vgain', 'class'])
df.vgain.value_counts()
```
_)_

Things have been hectic but starting to settle down with more free time now.

Looked at the question again ("How many samples in this dataset contain the value E for both motor and screw features?"), still didn't know how to do it and was pretty inefficient flipping through the notes in the course. It'd be better with actual documentation listed by function or something. Thus, I consulted the resources in the "Dive Deeper". 

Anyhow, I figured it out (and realized I didn't read the notes close enough, lol.)

Basically to find the samples I needed to do

```python
df[(df['motor'] == 'E') & (df['screw'] == 'E')]
```
Which gave me

```
34      E     E      3      2  1.100000
85      E     E      4      3  0.843755
131     E     E      6      5  0.281251
139     E     E      4      2  0.506252
150     E     E      3      1  0.700001>
```

However I had to count by hand, which seemed pretty stupid and definitely not how to solve the problem. Tired of flipping through the notes, I consulted DuckDuckGo. 

Reference: [this question](https://stackoverflow.com/questions/15943769/how-to-get-row-count-of-pandas-dataframe#15943975)

```python
e2 = df[(df['motor'] == 'E') & (df['screw'] == 'E')]
len(e2)
```
Which gave me `6`. (Originally I was confused and renamed the first row to the headers which ended up giving me `5`.)

Anyhow, onto the 3rd question.

**mean vgain value of those samples that have a pgain feature value equal to 4**

Pretty straightforward.

```python
vpgain = df[df['pgain'] == 4]
vpgain['vgain'].mean()
```

```
2.0606060606060606
```

### Lab Assignment 4

**Load up the table into a dataframe**

```python
df = pd.read_html('http://www.espn.com/nhl/statistics/player/_/stat/points/sort/points/year/2015/seasontype/2', header=1)[0]
```

I ended up installing `html5lib` first, couldn't fix the header unless I specified as above (really struggled with that = =), and also didn't realize how important `[0]` was. Apparently from the docs:

>If you print the result of your `read_html()` method, you will see that it's a list with a Pandas dataframe in it. To access the 0th element of the list, use: `dataframes = pd.read_html('...'); my_dataframe = dataframes[0]`


**Rename the columns so that they match the column definitions on the website**

Ha, so _now_ we rename... But I already did it because I, for some reason, could not rename the columns.

Nonetheless, it looked like there were some rows with labels, so I dropped them as duplicates

```python
df = df.drop_duplicates(subset=['RK','PLAYERS','TEAM'])
```

Also manually went and dropped the extra row with the labels.

**Get rid of any erroneous rows that has at least 4 NaNs in them**

There are 16 columns, which means we want to have rows with at least 12 non-NaNs

```python
df = df.dropna(axis=0, thresh=12)
```

**Get rid of the RK column**

```python
df = df.drop(labels=['RK'], axis=1).reset_index(drop=True)
```

**Ensure there are no nan "holes" in your index. Check the dtypes of all columns, and ensure those that should be numeric are numeric**

Honestly this was a pain in the ass to implement because there were so many columns. I wanted to find if there was a way to batch edit those things but it was pretty late by the time I got to this point and I just wanted to go to sleep because I have to wake up at 6:15am and it was around 11:40pm when I got to that point.

All in all I finally completed the assignment and got the answers right. Phew!

_Note: Just trying to update this blog took me a while...I had issues with gem dependencies and I'll figure out how to update to later versions of json later..._

