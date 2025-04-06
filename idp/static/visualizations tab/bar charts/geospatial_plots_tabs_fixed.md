
# Geospatial Plots

Geospatial plots are great at:  
1. Showing how an **area** is impacted relative to other **areas.**  
2. Revealing which area is impacted.  
3. Illustrating a pattern in the location of something.

Geospatial plots are NOT good at:  
1. Showing **how** one value correlates to another. (Use a line plot for this)  
2. Emphasizing the amount of difference between one area and another.  

## Contiguous States Only

{% tabs contiguous %}

{% tab contiguous image %}
One the left, we see what many students show--the full 50 states.  
One the right, is a much improved image showing only the 48 contiguous states.  
![election image](../_static/geo_demo_clip.jpg)
{% endtab %}

{% tab contiguous code %}
```python
fig, (ax1, ax2) = plt.subplots(1,2, figsize=(15,5))

# Left Side: Plot the entire United States
gdf.plot(ax=ax1, column='Electoral Votes')

# Right Side: Plot the contiguous US with a legend
gdf.plot(ax=ax2, column='Electoral Votes', legend=True)
ax2.axis('off') # remove the box and the x,y ticks

# Set the extent of the axes to cut off Alaska and Hawaii
ax2.set_xlim(-130, -65)
ax2.set_ylim(24, 50)
```
{% endtab %}

{% tab contiguous data %}
This data is a merge of two different data sources: electoral_college.csv and a ShapeFile of the United States.

![election data](../_static/geo_elect_data.jpg)

```python
electoral = pd.read_csv('electoral_college.csv', thousands=',')
gdf = us_states.merge(electoral, left_on='NAME', right_on='State', how='left')
gdf = gdf.drop(columns='State') # this is a duplicate of 'NAME'
```
{% endtab %}

{% tab contiguous comments %}
In this example, we have two geospatial in one figure where there are just three differences.  
1. The box and ticks are hidden  
2. We clip off Alaska & Hawaii  
3. We show a legend  

By hiding Alaska and Hawaii, we can bring a much greater focus on the states. Providing a legend is very helpful. And, eliminating the box with longitude and latitude ticks removes an unnecessary distraction.

In the chart on the right, California is clearly emphasized. You can hide Alaska & Hawaii by filtering:
```python
gdf = gdf[(gdf['NAME'] != 'Alaska') & (gdf['NAME'] != 'Hawaii')]
```
Or by setting axis limits:
```python
ax2.set_xlim(-130, -65)
```
{% endtab %}

{% endtabs %}
