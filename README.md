
### Introduction: Just a bit better

In the last section, we saw our process for improviding regression lines.  We started with a simple regression line of the form, $\overline{y}= \overline{m}x + \overline{b} $ to predict an output, given an input.  Then, we defined our errors to be the differences between the outputs that our regression line predict and the actual values.

![regression-scatter.png](./regression-scatter.png)

We then quantify how well our regression line maps to our data by squaring all of the errors (to eliminate negative values) and adding these squares together, to get our residual sum of squares (RSS).  Armed with a number that describes "goodness of fit", we iteratively try new regression lines by adjusting our y-intercept value, $b$, or slope value, $m$, and assessing goodness of fit.  By finding the values $m$ and $b$ that minimize the RSS, we can find our "best fit line".  In our cost function below, you can see the sequential values of $b$ and the RSS that these values produce (given a specific value $m$).  The bottom of the blue curve displays the $b$ value that produces the lowest RSS.


```python
import plotly
from plotly.offline import init_notebook_mode, iplot
from graph import m_b_trace, trace_values, plot, build_layout
init_notebook_mode(connected=True)
b_values = list(range(70, 150, 10))
rss = [10852, 9690, 9128, 9166, 9804, 11042, 12880, 15318]

layout = build_layout(options = {'title': 'RSS with changes to y-intercept'})
cost_curve_trace = trace_values(b_values, rss, mode="line")
plot([cost_curve_trace], layout)
```


<script>requirejs.config({paths: { 'plotly': ['https://cdn.plot.ly/plotly-latest.min']},});if(!window.Plotly) {{require(['plotly'],function(plotly) {window.Plotly=plotly;});}}</script>



<div id="9d3df4ad-d36c-4350-9219-481739b4c6d6" style="height: 525px; width: 100%;" class="plotly-graph-div"></div><script type="text/javascript">require(["plotly"], function(Plotly) { window.PLOTLYENV=window.PLOTLYENV || {};window.PLOTLYENV.BASE_URL="https://plot.ly";Plotly.newPlot("9d3df4ad-d36c-4350-9219-481739b4c6d6", [{"x": [70, 80, 90, 100, 110, 120, 130, 140], "y": [10852, 9690, 9128, 9166, 9804, 11042, 12880, 15318], "mode": "line", "name": "data", "text": []}], {"title": "RSS with changes to y-intercept"}, {"showLink": true, "linkText": "Export to plot.ly"})});</script>


Now, we don't want to try **all** of the different values for a regression line, $m$ and $b$.  Doing so takes too long, as there are too many possibilities. Especially as if our regression formulas will only get more complicated.  This technique doesn't work.

So in the last lesson, instead of checking all of the values, we evaluated increments of 10 to see if our changing $b$ value produced a higher or lower RSS.  

| b        | residual sum of squared           | 
| ------------- |:-------------:| 
| 140 | 15318 | 
| 130      |12880| 
| 120      |11042 | 
| 110      |9804| 
|100 | 9166
|90 | 9128
|80 | 9690
|70| 10852

Looking at the chart above, we settled in on a value of $b$ between 90 and 100, as that is where produced the corresponding lowest RSS.

> Remember, that ultimately we want to change both of our parameters to find the "best fit" line, but for now we just change one to make things easier.

### Still, things are not so simple

The above approach of checking our RSS in with varying our $b$ value by 10 works pretty well.  The only thing we want to differently is changing our value of $b$ more carefully such that we **know** our change will continue to reduce our RSS.  Adjusting the y-intercept value and then checking the effect is a bit like trying to fly plane by just moving sitting down and pressing buttons.  Is there a way that even with our first change, we can rest assured we're moving in the right direction?  And how do we know how large a **change** to make to our y-intercept after each calculation of RSS?  

Let's call each of these changes a step, and the size of the change our step size.  So our new task is to have our step sizes get us to our RSS quickly and without overshooting the mark.

![](https://bossip.files.wordpress.com/2014/11/aden-and-cree-580x435.jpg)

### The slope of the cost curve tells us our step size

Believe it or not, we can determine how large our step size should be looking the slope of our cost function.

To see this, let's zoom in on our cost function, and look at just one part of it.  Looking at our zoomed in cost function below, do you see a way that we can get a sense of the direction and size of our next step?  


```python
import plotly
from plotly.offline import init_notebook_mode, iplot
from graph import m_b_trace, trace_values, plot
init_notebook_mode(connected=True)
b_values = list(range(70, 150, 10)[:3])
rss = [10852, 9690, 9128, 9166, 9804, 11042, 12880, 15318][:3]
cost_curve_trace = trace_values(b_values, rss, mode="line")
plot([cost_curve_trace])
```


<script>requirejs.config({paths: { 'plotly': ['https://cdn.plot.ly/plotly-latest.min']},});if(!window.Plotly) {{require(['plotly'],function(plotly) {window.Plotly=plotly;});}}</script>



<script>requirejs.config({paths: { 'plotly': ['https://cdn.plot.ly/plotly-latest.min']},});if(!window.Plotly) {{require(['plotly'],function(plotly) {window.Plotly=plotly;});}}</script>



<div id="105abb88-c470-4a0c-98b2-09d1ad798059" style="height: 525px; width: 100%;" class="plotly-graph-div"></div><script type="text/javascript">require(["plotly"], function(Plotly) { window.PLOTLYENV=window.PLOTLYENV || {};window.PLOTLYENV.BASE_URL="https://plot.ly";Plotly.newPlot("105abb88-c470-4a0c-98b2-09d1ad798059", [{"x": [70, 80, 90], "y": [10852, 9690, 9128], "mode": "line", "name": "data", "text": []}], {}, {"showLink": true, "linkText": "Export to plot.ly"})});</script>


Umagine yourself standing on your cost curve.  Even with your eyes closed, you could tell simply by the way you were tilting whether to walk forwards or backwards to approach the bottom of the cost curve.  If the slope tilts downwards at that point  we walk forward, and if the slope tilts upwards at that point we walk backwards.  Then the steeper the tilt, the larger the step we should take as the further away we are from our cost curve.  

So by looking to the tilt of a cost curve at a given point, we can discover the direction of our next step and how large of step to take.  

### Stepping according to the slope

![](./cost-chart-slope.png)

We can follow our technique with more precision by adding some numbers to our slope.  The slope of curve at any given point is equal to the slope of the tangent line at that point.  By tangent line, we mean the line that just barely touches the curve at that point.  In the above graph, the orange, green, and red lines are tangent to our cost curve at the points where $b$ equals 70, 84, and 90 respectively.  The slopes of our tangent lines, and therefore the slopes of the cost curves at those points, are labeled above.  

The whole point of looking at the slope is because it supposedly tells us the size and direction of our next step, and thus tells us how to change our value of $b$.  Let's see how this works.

We can use the following procedure for approaching our $b$ finding the ideal $b$: 
1.  Randomly choose a value of $b$, and then 
2.  Update $b$ with the formula $ b = (-.1) * slope_{b = i} + b_i$.

All that formula says, is choose a value of $b$, and then move it to be a small number, -.1 times the slope of the tangent line at that point.  This way, the larger the slope, the larger the step.  And the negative means we will always move in the opposite direction of the slope (that is, when the tangent line points downwards move forwards). 

Let's see an example.  We randomly choose our value of $b$ to equal 70.  Then:

* $b_{t=0} = 70 $
* $b_{t=1} = (-.1) * -146.17  + 70 = 14.61 + 70 = 84.61 $
* $b_{t=2} = (-.1) * -58.51 + 85 = 5.851 + 85 = 90.851 $
* $b_{t=3} = (-.1) * -21.07 + 90.85 = 90.851 + 2.107 $

So notice that we don't update our values of $b$ by just adding or subtracting the slope at that point.  Doing so would be too drastic.  Instead we multiply by the slope by a fraction -- in this case -- $.1$ which is called a **learning rate**.  This way, the steeper slope of the tangent line, the more we change in $b$, but we still make sure we are not changing our regression lines too drastically.  
> We should point out that, in general, learning rates are much smaller than what we saw here (.001 is more typical), but we used this learning rate just for demonstration purposes.   

Also notice that this technique is pretty magical.  By looking at the tangent line at each point, we no longer are  changing our $b$ value and just hoping that it has the correct impact on our RSS.  This is because, for one, the slope of the tangent line points us in the right direction.  And as you can see above, our technique properly adjusts the amount to change the $b$ value by without even knowing where ideal $b$ value is.  When our $b$ was far away from the ideal $b$ value our formula had us increase our $b$ by 14, and in just three steps we were only updating our $b$ value by 2, as we approached the $b$ that minimizes our RSS.  

### Summary

We started this section with saying that we wanted a technique to find a $b$ value that would minimize our RSS, given a value of $m$.  We did not want to simply try all of the values of $b$ as doing so would be inefficient.  Instead, we went with the approach of gradient descent, where we try variations of regression lines by iteratively changing our $b$ variable and assessing our RSS to see if we are making progress.

In this lesson, we focused in on how to know which direction to alter a given variable, $m$ or $b$, as well as a technique for determining the size of the change to one of our variables.  We used the line tangent to our cost curve at a given point to indicate the direction and size of the update to $b$.  The further away, the steeper the curve and thus the larger the step we would want to take.  Appropriately, our tangent line slope would have us take a larger step.  And the closer we are to the ideal $b$ value, the flatter the tangent line to the curve, and the smaller a step we would take. 
