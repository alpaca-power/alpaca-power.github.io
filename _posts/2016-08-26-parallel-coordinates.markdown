---
title:  "Notes on Module 3"
date:  2016-08-26 11:37:00
category: [python]
tags: [python]
---

Today I went through Module 3. This time the lab exercises were easy, but I had a hard time understanding parallel coordinates.

In particular, looking at this:

```python
data = load_iris()
df = pd.DataFrame(data.data, columns=data.feature_names) 

df['target_names'] = [data.target_names[i] for i in data.target]

# Parallel Coordinates Start Here:
plt.figure()
parallel_coordinates(df, 'target_names')
```

Only made me confused. What the heck was going on? Didn't understand... I knew this `df['target_names']` was iterating through something, but I had no clue why or what that was. I had no idea why the second input for `parallel_coordinates` was `'target_names'`, so I didn't know how to make a parallel coordinate thing for the lab exercies.

Searched the [official documentation](http://pandas.pydata.org/pandas-docs/stable/visualization.html#parallel-coordinates) for any clues on why this is done. Apparently they were also using the iris dataset for the example, except now it was: 

```python
In [89]: from pandas.tools.plotting import parallel_coordinates

In [90]: data = pd.read_csv('data/iris.data')

In [91]: plt.figure()
Out[91]: <matplotlib.figure.Figure at 0x11fb7fa50>

In [92]: parallel_coordinates(data, 'Name')
Out[92]: <matplotlib.axes._subplots.AxesSubplot at 0x121372590>
```

Well, well...this was...only even more confusing. What the heck was `'Name'` this time?!

Consulted DDG (which of course led me to SO). Found a [somewhat clearer explanation](https://stackoverflow.com/questions/38103738/plotting-parallel-coordinates-in-pandas-python/38104166), which said you should put in the 'class', but still felt somewhat confused. `'Name'`?!

And finally, somehow stumbled upon someone posting the actual csv used [here](https://raw.githubusercontent.com/pydata/pandas/master/pandas/tests/data/iris.csv). Now it made more sense! The class column name, of course! 

It also dawned on me I was plotting the wrong thing (the dataframe which I didn't drop features off), no wonder the whole plot was out of whack with scaling issues.

Anyhow. I feel like I understand parallel coordinates a bit more now. After loading the actual iris datasest from `sklearn`, I finally saw why the heck was the edX example so complex. They had to build a dataframe from the whole thing since it wasn't as neat as the csv example above.

Other source I found pretty helpful: [https://eagereyes.org/techniques/parallel-coordinates](https://eagereyes.org/techniques/parallel-coordinates) 

In particular, this line was a nice emphasis on what not to do:

>The number of cylinders can only be a whole number, and there aren’t more than eight here, so all the lines have to pass through a small number of points. Data like this, and also categorical data, are usually not well suited for parallel coordinates. As long as there is only one or two, it’s not a problem, but when the data is largely or completely categorical, parallel coordinates do not show any useful information anymore

(apparently it was written by a senior research scientist at Tableau, nice content marketing there ha!)

#### Additional Notes

`fig.add_subplot(111)` also confused me. Why `111`? Naturally I looked at the official docs, and lo and behold there was also something like `fig.add_subplot(212)`. (╯‵□′)╯︵┻━┻

Really thought they should explain more in the course's notes, or at least link things to the official docs to make searching easier. Anyhow, DDG/SO to the rescue again.

Glad to find I'm [not the only one](https://stackoverflow.com/questions/3584805/in-matplotlib-what-does-the-argument-mean-in-fig-add-subplot111) confused. Very helpful, along with the link in the 3rd answer. The first answer I sort of got and sort of didn't, I mixed it up with the 2nd answer's image which was what initially threw me off (kept wondering why the graphs were the same size, lol). But yeah I also experimented a bit on my iPython console so 'twas all good.

Additionally, [MatPlotLib's docs](http://matplotlib.org/api/axes_api.html#matplotlib.axes.Axes.scatter) also helped. When I initially saw `c='r'` and `marker='.'` I did not know what those were for. Of course, it's pretty obvious now, but yeah. Glad I got that cleared up.


#### TODO:
Also find time to do [Numpy practice questions](http://www.labri.fr/perso/nrougier/teaching/numpy.100/), because the really popular Python for DataScience Udemy course I bought was not helpful. In fact, it was super boring and was also a pain to go through...felt like a frickin' chore and made me question whether I could really learn the stuff.

The problem was, of course, it was all talk and no useful exercise to work on. Pretty much forgot everything about numpy by now, but can probably better learn it through doing exercises.

Speaking of which, I should get on doing CodeChef problems too, because I haven't touched basic Python after I finished the Codecademy course, and I can barely remember how to do basic things. 

Alas, so many things (including MBA classes and the clubs that will be starting soon). So little time. Just hope I can keep this learning momentum going.
