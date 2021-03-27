---
layout: posts
title:  "Data engineering with Python"
categories: coding
date: 2021-03-27 21:06:00 +0100
---

In the first wave of the covid-19 pandemic I came across a site called [DataCamp][datacamp]. They offered a
huge discount on online data science courses mainly focusing on matplotlib and pandas framework. So I jumped in. Pandas is a great tool handling large databases in json, csv, sql, or even excel. You can easily analize, them, and extract the desired part. With matplotlib we can do visualisations of such data streams.
  As I finished the tutorial, It was quite obvious to use it for analizing the current trends of the pandemic.That way I could test and practice my new skills. By the way this sort of helped dealing with the lockdown situation psychologically too.
Let's see how it goes. I am not gonna go in detail on how to install Python packages. It can be very tricky, and there are many ways doing it. Easiest solution is to make a new Python project in an IDE (Integrated Development Environment) such as Pycharm for example. There you create a new Python file and simply add: {% highlight python %}  
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
{% endhighlight %}
 ,then click on the error next to the code to install the packages. It is as easy as that. The rest of the heavy-lifting is handled by the IDE. 
 Next thing we need is a data source. You can search Github for any kind of data APIs, just make a search on the desired data, and choose the best one for you. I stuck with the Our World In Data [website's][owid] data in comma separated values. So I went: {% highlight python %} data = pd.read_csv("https://raw.githubusercontent.com/owid/covid-19-data/master/public/data/owid-covid-data.csv") {% endhighlight %}
Next you have to analize the data somehow. Try to look inside. In the 
{% highlight python %} print(data.head()) {% endhighlight %}
That will try to print out the header and the first five rows of the data. We can see that the rows are representing the country datas column by column, and each country has rows for each day passed since the outbreak of the pandemic. The problem is, this data is too huge to see all the columns. Try to get all the data we have for each country:  {% highlight python %}  for c in data:
  print(c) {% endhighlight %} Now you get all the column names. Just out of curiosity, count the columns. let's make a counter variable and iterate through them. {% highlight python %} 
  counter = 0
  for c in data:
      counter += 1
{% endhighlight %}
Then:
{% highlight python %}
print(counter)
//61
{% endhighlight %}
Now we know that we are dealing with 61 columns of data. How we know, how many entries are there for all the countries? It is easier just print out the shape of the database. 
{% highlight python %}
print(data.shape)
//(77549, 61)
{% endhighlight %}
Ok, now we know there are 61 columns for 77549 rows. Makes sense, because that includes more than a year of daily data for each country on Earth. We can clearly see, that dealing with a data of that scale would be almost impossible without writing scripts. Not to mention visualising it.
Without going in all the details you can do with these frameworks, let's just see a very simple sample  visualizing some segment of our data. When we printed out all the columns, we saw that there is one column for location. That means we can filter this whole database for one countries data.
{% highlight python %} 
uk_data   = data[data['location'] == 'United Kingdom']
{% endhighlight %}

One interesting column have the name new_cases. That is representing the daily registered new infections.
  Let' get that:

{% highlight python %}
new_cases = uk_data['new_cases']
{% endhighlight %}

In order to be able to visualise this data, we need to tell pyplot, what are the x values for the chart. That is gonna be the axis for the days passed. In other words that is gonna be an array of dates.
{% highlight python %}
dates = uk_data['date']
{% endhighlight %}

Now we have the necessary variables for our chart. Add
{% highlight python %}
plt.plot(uk_dates, new_cases)
plt.show()
{% endhighlight %}
 At this point our code looks like this.
{% highlight python %}
import pandas as pd 
import matplotlib.pyplot as plt 
import numpy as np 

data = pd.read_csv("https://raw.githubusercontent.com/owid/covid-19-data/master/public/data/owid-covid-data.csv")

uk_data = data[data['location'] == 'United Kingdom']
new_cases = uk_data['new_cases']
uk_dates = uk_data['date']

plt.plot(uk_dates, new_cases)
plt.show()
{% endhighlight %}

Run the project. You are going to see a plot chart of the daily covid-19 cases in the UK.
The dates below the chart are just a black line. This is because the dates are written out in yyyy-MM-dd form, and there are too big number of them. Let's solve the problem by diplaying only every n-th date.
{% highlight python %}
sample_frequency = 11
uk_dates = uk_data['date'][:-1: sample_frequency]
{% endhighlight %}
This means we want all the entries to the last one, by step dfined in the sample_frequency variable. Currently we set it to step by 11 days.
Let's display a shorter version of the date strings and store them in an array. This can be solved by writing a function called trimmedDates.
{% highlight python %}
def trimmedDates(array):
    resultArray = []
    for date in array:
        date = date[-5:]
        resultArray.append(date)
    return resultArray
{% endhighlight %}

This function is taking an array argument and returning an array of date strings, 5 digits each. We can use them in the xtics function like the following:

{% highlight python %}
plt.plot(uk_data['date'], new_cases)
plt.xticks(np.arange(0, len(uk_data['date']), sample_frequency), trimmedDates(uk_dates))
{% endhighlight %}
Note: we are using all the uk dates in the plot function, we only filtered for the x tick labels as uk_dates. Let's run the project now. Better, but the dates are still barely visible. Lets rotate them, and add them a smaller fontsize!
{% highlight python %}

plt.xticks(np.arange(0, len(uk_data['date']), sample_frequency), trimmedDates(uk_dates), fontsize=5, rotation=75)
{% endhighlight %}

At this point our code should look something like this:
{% highlight python %}
import pandas as pd 
import matplotlib.pyplot as plt 
import numpy as np 

data = pd.read_csv("https://raw.githubusercontent.com/owid/covid-19-data/master/public/data/owid-covid-data.csv")

uk_data = data[data['location'] == 'United Kingdom']
new_cases = uk_data['new_cases']
uk_data['date']

sample_frequency = 11
uk_dates = uk_data['date'][:-1: sample_frequency]

def trimmedDates(array):
	resultArray = []
	for date in array:
		date = date[-5:]
		resultArray.append(date)
	return resultArray

plt.plot(uk_data['date'], new_cases)
plt.xticks(np.arange(0, len(uk_data['date']), sample_frequency), trimmedDates(uk_dates), fontsize=5, rotation=75)
plt.show()

{% endhighlight %}
Run the project, and you should get something like the following: ![article_plot]({{baseUrl}}/assets/img/article_plot.png)
Thank You for reading! Happy Coding!


[datacamp]: https://www.datacamp.com
[owid]: https://ourworldindata.org