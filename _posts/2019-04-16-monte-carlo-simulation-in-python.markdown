---
layout: post
title:  "Monte Carlo Simulation in Python"
date:   2019-04-16 18:30:46 -0500
categories: python tutorials
---

Introduction
---------------

A [Monte Carlo Simulation](https://en.wikipedia.org/wiki/Monte_Carlo_method) is a method used for risk assessment
in project management, finance and general forecasting.  This technique is an effective way to simulate potential 
outcomes of a given decision, and is particularly useful when the analysis is based on estimated values.

I originally developed a Monte Carlo program out of necessity. I was working on an engineering risk analysis and couldn't 
find a good tool to perform this technique.  I found confusing Excel plug-ins and expensive dedicated software.  So I decided
to create my own Monte Carlo program!

In this article, we will run a simple Monte Carlo simulation to determine if a new manufacturing line is a good investment.
Rather than guessing if we will break even, let's use statistics!

Let's Get Started
--------------------


Prepare the Data
======================

A potential client requests 90,000-120,000 bottles per year of a new specialty bottled drink and wants to pay no more than
$1.75 per bottle.  We want to know if we should invest in a new manufacturing line to fulfill this request.  Our company 
likes payback periods to be under 2 years for new projects. Can we make our money back on this project in 2 years?

We found quotes for equipment, installation and ingredient costs, and our data is as follows:
- Capital Cost:		$300,000
- Operating Cost:	$1.00/bottle
- Production:		90,000-120,000 bottles/year
- Selling Price:	$1.75/bottle (maximum)

Now, we don't really *know* that the capital cost of the project will be $300,000 or that the drink will cost $1.00/bottle to
manufacture.  These are our expected *averages*, and any profit calculations we do with these numbers will be subject to 
error.  The real world is not always average.  This is where Monte Carlo comes in handy.

To prepare this data for a Monte Carlo simulation, we should come up with ranges for each uncertain category such that we 
are 90% sure the actual value will fall somewhere within the range.  Let's use the following ranges:
- Capital Cost:		$250,000-350,000
- Operating Cost:	$0.80 - $1.20/bottle
- Production:		90,000 - 120,000 bottles/year



Develop the Monte Carlo Program
===============================

First thing's first, we need to import pandas, norm, pyplot and statistics.

``` python
import pandas as pd
from scipy.stats import norm
import matplotlib.pyplot as plt
import statistics
```
Now it's time to flex our statistics knowledge.  We want to convert our expected values and 90% ranges into *normal
distributions*.  Remember that 90% of normally distributed data is found within +/- 1.645 standard deviations of the
mean, so this means that there are 2*1.645 = 3.29 standard deviations within our ranges.  Other values could be used
for this variable if we wanted to use a different confidence level.


To generate these normal distributions, we can use the following code:

``` python
num_stdev = 3.29

capital_cost = norm(loc = 300000, scale = (350000-250000)/num_stdev)
operating_cost = norm(loc = 1, scale = (1.20-0.80)/num_stdev)
production = norm(loc = 105000, scale = (120000-90000)/num_stdev)
```

Now it's time for the simulation.  We want to randomly generate values from these distributions so that they can be 
used to calculate net profit.  Let's generate one million values for each category.

``` python
num_simulations = 1000000

capital_cost_values = capital_cost.rvs(num_simulations)
operating_cost_values = operating_cost.rvs(num_simulations)
production_results = production.rvs(num_simulations)
```
And let's put these results into a dataframe called "data":

``` python
data = pd.DataFrame({
    "capital_cost": capital_cost_results,
    "operating_cost": operating_cost_results,
    "production": production_results,
    "benefits": production_results * 1.75,
})
```
*Notice that the "benefits" column is calculated by multiplying production results (bottles/year) by the selling price
($1.75/bottle).* 

The top 5 rows of our data should look something like this, though your randomized data may look a bit different:

![screenshot](/photos/data1.PNG){:class="img-responsive"}

Excellent.  Now let's create a new column to show net profit for each simulation.

``` python
data["net_profit"] = (data.benefits*2 - data.capital_cost - data.operating_cost*data.production*2)
```

*Notice that the net profit is calculated as total benefits - capital cost - total operating costs.*
- Total Benefits: benefits ($/year) * desired payback (years)
- Total Operating Costs: operating costs ($/bottle) * production (bottles/year) * desired payback (years)

We now have a net profit column that holds values corresponding to each row in our dataframe. This dataframe should
have one million rows.

![screenshot](/photos/data2.PNG){:class="img-responsive"}

Let's visualize net_profit.

``` python
plt.hist(data.net_profit, bins=100)
plt.xlabel("net profit ($)")
plt.show()
```
![screenshot](/photos/hist1.PNG){:class="img-responsive"}


Interpret the Results
===============================

Let's calculate the mean and standard deviation of net_profit.  We can then use these values to calculate the 
z-score of the break-even point (zero net profit).

``` python
stdev = statistics.stdev(data.net_profit)
mean = statistics.mean(data.net_profit)
z_score = (mean)/stdev
```
Using norm.cdf() we can convert this z-score to a probability.

``` python
print("The estimated probability of breaking even after 2 years: " + str(norm.cdf(z_score)))
```
Here is my output:

![screenshot](/photos/montecarlo4.PNG){:class="img-responsive"}

Notice that approximately 57% of the data in the histogram is to the right of zero.  This means that 57% of our 
simulations were profitable in the specified payback period.


What to Tell Our Boss
===============================

We can say that if we sell the product at $1.75/bottle, we estimate a 57% chance of breaking even in
2 years. (This might not sound very attractive to our boss.)  

However, if we run the simulation again with a 3 year payback, we estimate a 95% chance of breaking even. 
(Our boss might like the sound of that!)  

Accepting a longer payback may even allow us to lower our price from $1.75 in order to ensure customer satisfaction. 
(Try running the simulation with a selling price of $1.60/bottle and a 3 year payback period.)


Conclusion
---------------

As you can see, Monte Carlo simulation is a powerful tool for assessing risk by predicting future outcomes. 
There are many applications for this technique, and this post highlights just one example. This type of analysis
can be performed in Excel or with other tools, but programming it in Python is easy and fast. 

I hope this post provides you with a basic understanding of Monte Carlo theory and gives you ideas for how to do 
your own analysis. Maybe you can account for depreciation and interest. Feel free to email me and let me
know what you're working on, or your thoughts on this article!  I am working on implementing a comment feature
to this website.

Also, feel free to download this code from my GitHub and use/edit it for your own project.