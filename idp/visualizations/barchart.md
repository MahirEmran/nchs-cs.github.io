---
layout: default_dl
title: Bar Charts
parent: Visualizations
nav_order: 1
---

# Bar Charts

Bar charts are good when:  
* The x-axis is categorical and lacks a natural order (e.g., country names).  
* There are few points to show (e.g., 4 or fewer).  
* There’s no obvious relationship to the prior x-axis value (due to categorical nature).  
* You want to use color to emphasize height or show a relationship to another value.  
* You have a count to represent.  

Bar charts are NOT good when:  
* You’re trying to show a correlation (e.g., more X means more Y).  
* The data is continuous, not discrete.  
* There are many points to show (e.g., 20 or more).

Here’s a summary of APIs for creating bar charts, with details in the examples below:  

| API                   | When to Use                                                                 |
|-----------------------|-----------------------------------------------------------------------------|
| `df.plot(kind='bar')` | One value per category.                                                    |
| `df.plot.barh()`      | One value per category, horizontal bars add meaning or many categories.    |
| `plt.bar()`           | Similar to `df.plot`, but useful without a DataFrame.                      |
| `plt.hist()`          | You want to count occurrences.                                             |
| `sns.barplot()`       | Many values per category to average; flexible data structure.              |
| `sns.catplot(kind='bar')` | Effectively the same as `sns.barplot()`.                               |

## Positive vs Negative Bars
This bar chart uses green for positive values and red for negative ones, emphasizing direction of change. Despite many years, the color distinction justifies its use.

{% tabs pos_neg %}

{% tab pos_neg Image %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/annual_change.png)
{% endtab %}

{% tab pos_neg Code %}
```python
def pos_neg_bar_chart(df):
    fig, ax = plt.subplots(1)
    df['color'] = df['Annual Change'].apply(lambda v: 'green' if v >= 0 else 'red')
    df.plot(kind='bar', x='date', y='Annual Change', width=0.8, color=df['color'], ax=ax, label='Positive Change')
    plt.xticks(np.arange(0, 61, 10))
    plt.xlabel('Year')
    plt.ylabel('Change in %')
    plt.title('Annual Change in Inflation from 1961-2021')
    fig.autofmt_xdate()
```
{% endtab %}

{% tab pos_neg Data %}
First few lines of the DataFrame:  
![Bar Chart](../static/visualizations%20tab/bar%20charts/inflation_data.jpg)
{% endtab %}

{% tab pos_neg Comments %}
* `plt.xticks(np.arange(0, 61, 10))` reduces x-axis ticks for readability.  
* `fig.autofmt_xdate()` slants dates; alternatively, use `plt.xticks(rotation=45)`.  
* “Change in” in the title and y-label indicates relative change, not absolute inflation rates (typically positive).
{% endtab %}

{% endtabs %}

## Overlayed Bars
This chart overlays bars (not stacked), with taller bars plotted first. It includes annotations for a calculated “payback” value.

{% tabs overlayed %}

{% tab overlayed Image %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/annotated_bars.png)
{% endtab %}

{% tab overlayed Plot Code %}
```python
def annotated_bars(df):
    df['payback'] = df['return'] / df['investment']
    fig, ax = plt.subplots(1, figsize=(15, 5))
    plt.bar(x=df['year'], height=df['return'], color='#10E0D0', label='Return')
    add_value_labels(ax, df['payback'], fmt='x{:.2f}')
    plt.bar(x=df['year'], height=df['investment'], color='#A030A0', label='Investment')
    plt.xlabel('Year')
    plt.ylabel('Dollars')
    plt.xticks([int(y) for y in df['year']], rotation=-45)
    plt.legend()
    plt.title('Investments & Returns')
```
{% endtab %}

{% tab overlayed Annotation Code %}
Source: [StackOverflow](https://stackoverflow.com/questions/28931224/how-to-add-value-labels-on-a-bar-chart)  
```python
def add_value_labels(ax, values, fmt="{:.1f}", spacing=2):
    """
    Add labels to the end of each bar in a bar chart.
    Arguments:
        ax (matplotlib.axes.Axes): The axes object to annotate.
        values: Sequence of values to show.
        fmt: Format string for labels.
        spacing (int): Pixel distance between labels and bars.
    """
    for index, rect in enumerate(ax.patches):
        y_value = rect.get_height()
        x_value = rect.get_x() + rect.get_width() / 2
        space = spacing
        va = 'bottom'
        if y_value < 0:
            space *= -1
            va = 'top'
        label = fmt.format(values[index])
        ax.annotate(label, (x_value, y_value), xytext=(0, space), textcoords="offset points", ha='center', va=va)
```
{% endtab %}

{% tab overlayed Data %}
Complete DataFrame (payback calculated in code):  
![Bar Chart](../static/visualizations%20tab/bar%20charts/annotation_data.jpg)
{% endtab %}

{% tab overlayed Comments %}
* Annotation code is separated due to complexity.  
* Overlay achieved by plotting taller bars first, then shorter ones atop.  
* Annotations use `ax.patches` for bar locations, added after the first plot to avoid overlap issues.  
* Custom colors use `#RRGGBB`; figure size increased for readability.
{% endtab %}

{% endtabs %}

## Sorted Bars
Bar charts typically lack a natural x-axis order, but sorting by height offers insight.

{% tabs sorted %}

{% tab sorted Image %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/sorted_bars.png)
{% endtab %}

{% tab sorted Code %}
```python
def sorted_plt_bars(df):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    ax.grid(False)
    df = df.sort_values(by='a_value', ascending=False)
    bar = plt.bar(x=df['state'], height=df['a_value'])
    ax.bar_label(bar, label_type='center')
    plt.xticks(rotation=60)
    plt.title('Best "A Value" by State')
    plt.ylabel('A Value')
    plt.xlabel('')
```
{% endtab %}

{% tab sorted Data %}
First 11 rows of unsorted original data:  
![Bar Chart](../static/visualizations%20tab/bar%20charts/sorted_data.jpg)
{% endtab %}

{% tab sorted Comments %}
* Sorting aids viewers with many bars.  
* `ax.bar_label` adds values inside bars; alternatives include `plt.text` or `ax.annotate`.  
* X-axis label is empty as state names are self-explanatory.  
* Colors can be customized with `color` (single color or sequence via `df['color']`).
{% endtab %}

{% endtabs %}

## Adjusted Y-Label
Bar charts typically start at y=0. Adjusting the y-axis highlights differences. Three approaches are shown: using `bottom`, `ylim` with patches, and `edge` labeling.

{% tabs adjusted_y %}

{% tab adjusted_y Image 1 %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/bars_bottom.png)
{% endtab %}

{% tab adjusted_y Image 2 %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/bars_patches.png)
{% endtab %}

{% tab adjusted_y Image 3 %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/bars_edge.png)
{% endtab %}

{% tab adjusted_y Code 1 %}
```python
def sorted_plt_bars(df):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    ax.grid(axis='y')
    ax.set_axisbelow(True)
    df = df.sort_values(by='a_value', ascending=False)
    bottom = 40
    df['adj_value'] = df['a_value'] - bottom
    bar = plt.bar(x=df['state'], height=df['adj_value'], bottom=bottom)
    ax.bar_label(bar, labels=df['a_value'], label_type='center')
    plt.xticks(rotation=60)
    plt.title('Best "A Value" by State')
    plt.ylabel('A Value')
    plt.xlabel('')
    plt.yticks([y for y in range(40, 100, 10)], ['one', 'two', 'three', 'four', 'five', 'six'])
```
{% endtab %}

{% tab adjusted_y Code 2 %}
```python
def sorted_plt_bars(df):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    df = df.sort_values(by='a_value', ascending=False)
    df = df.set_index('state')
    ax = df['a_value'].plot(kind='bar', width=0.7, color='lightcoral')
    bars = ax.containers[0]
    bottom = 40
    bars.patches = [plt.Rectangle(r.get_xy(), r.get_width(), r.get_height() + bottom) for r in bars.patches]
    ax.grid(True, which='major', axis='both', color='b', ls=':')
    ax.set_axisbelow(True)
    plt.ylim(custom=bottom, top=100)
    ax.bar_label(bars, labels=df['a_value'], label_type='center')
    plt.xticks(rotation=60)
    plt.title('Best "A Value" by State')
    plt.ylabel('A Value')
    plt.xlabel('')
```
{% endtab %}

{% tab adjusted_y Code 3 %}
```python
def sorted_plt_bars(df):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    df = df.sort_values(by='a_value', ascending=False)
    df = df.set_index('state')
    ax = df['a_value'].plot(kind='bar', color='seagreen')
    bars = ax.containers[0]
    ax.grid(True, which='major', axis='both', color='y', ls='-.')
    ax.set_axisbelow(True)
    plt.ylim(custom=40, top=110)
    ax.bar_label(bars, labels=df['a_value'], label_type='edge')
    plt.xticks(rotation=60)
    plt.title('Best "A Value" by State')
    plt.ylabel('A Value')
    plt.xlabel('')
```
{% endtab %}

{% endtabs %}

## Stacked Bars
Stacking bars shows multiple values and their total, sorted for insight.

{% tabs stacked %}

{% tab stacked Image %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/stacked_bars.png)
{% endtab %}

{% tab stacked Code %}
```python
def stacked_df_plot_kind(df):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    df = df.sort_values(by='a_value', ascending=False)
    df = df.set_index('state')
    df.plot(kind='bar', stacked=True, ax=ax)
    plt.xticks(rotation=60)
    plt.title('Stacked Bar')
    plt.xlabel('')
    plt.ylabel('Both Values')
```
{% endtab %}

{% tab stacked Data %}
First 11 rows of unsorted original data:  
![Bar Chart](../static/visualizations%20tab/bar%20charts/sorted_data.jpg)
{% endtab %}

{% tab stacked Comments %}
* Sorting could be by total value, depending on intent.  
* Additional columns would stack; here, only two are plotted.
{% endtab %}

{% endtabs %}

## Side-by-Side Bars
Compare values across categories with side-by-side bars, shown via `plt` and `seaborn`.

{% tabs side_by_side_plt %}

{% tab side_by_side_plt Image %}
Similar to the Seaborn plot, with subtle differences (no legend title, opaque legend, vertical grid lines).  
![Bar Chart](../static/visualizations%20tab/bar%20charts/plt_side-by-side-bars.png)
{% endtab %}

{% tab side_by_side_plt Code %}
```python
def plot_side_by_side(df):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    df = df.sort_values(by='b_value', ascending=True).reset_index(drop=True)
    df = df.loc[:9]
    df = df.set_index('state')
    df.plot(kind='bar', stacked=False, width=0.8, ax=ax)
    plt.xticks(rotation=60)
    plt.title('Top 10 States')
    plt.ylabel('Both Values')
    plt.xlabel('')
    plt.legend(['A Value', 'B Value'])
```
{% endtab %}

{% tab side_by_side_plt Data %}
First 11 rows of unsorted original data:  
![Bar Chart](../static/visualizations%20tab/bar%20charts/sorted_data.jpg)
{% endtab %}

{% tab side_by_side_plt Comments %}
* Side-by-side needs space; limited to 10 states.  
* `width=0.8` adjusts bar feel.  
* `colormap='Spectral'` customizes colors; defaults used here.  
* `reset_index()` ensures slicing works reliably.
{% endtab %}

{% endtabs %}

{% tabs side_by_side_sns %}

{% tab side_by_side_sns Image %}
Nearly identical to `plt` plot above.  
![Bar Chart](../static/visualizations%20tab/bar%20charts/side-by-side-bars.png)
{% endtab %}

{% tab side_by_side_sns Data %}
Data after `pd.melt` (original in `plt` example):  
![Bar Chart](../static/visualizations%20tab/bar%20charts/bar_sns_melt_data.jpg)
{% endtab %}

{% tab side_by_side_sns Code %}
```python
def sns_side_by_side(df):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    df = df.sort_values(by='b_value', ascending=True).reset_index(drop=True)
    df10 = df.loc[:9]
    df10 = pd.melt(df10, id_vars='state', var_name='Value_Type', value_vars=['a_value', 'b_value'], value_name='Value')
    sns.barplot(data=df10, x='state', y='Value', hue='Value_Type')
    handles, labels = ax.get_legend_handles_labels()
    plt.xticks(rotation=60)
    plt.title('Top 10 States')
    plt.ylabel('Both Values')
    plt.xlabel('')
    plt.legend(handles=handles, title='Values', labels=['A Value', 'B Value'])
```
{% endtab %}

{% tab side_by_side_sns Comments %}
* `hue` leverages melted data structure.  
* `pd.melt` simplifies column-to-category transformation.  
* `reset_index()` ensures slicing consistency.  
* Legend customized with `handles` for correct colors and title.
{% endtab %}

{% endtabs %}

## Horizontal Bar Chart
Use `barh` for horizontal bars.

{% tabs horizontal %}

{% tab horizontal Image %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/horiz_bars.png)
{% endtab %}

{% tab horizontal Code %}
```python
def df_horiz_plot(df):
    fig, ax = plt.subplots(1, figsize=(8, 10))
    df = df.sort_values(by='state', ascending=False)
    df = df.set_index('state')
    df.plot.barh(stacked=False, width=0.6, ax=ax)
    plt.xticks(rotation=60)
    plt.title('State Values')
    plt.xlabel('Values')
    plt.ylabel('State')
    plt.legend(['A Value', 'B Value'])
```
{% endtab %}

{% tab horizontal Data %}
First 11 rows of unsorted original data:  
![Bar Chart](../static/visualizations%20tab/bar%20charts/sorted_data.jpg)
{% endtab %}

{% tab horizontal Comments %}
* DataFrame’s [plot object](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.plot.html) supports methods like `barh`, `bar`, `area`, etc.
{% endtab %}

{% endtabs %}

## Histogram
Histograms count occurrences with bars.

{% tabs histogram %}

{% tab histogram Image %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/distance_hist.jpg)
{% endtab %}

{% tab histogram Code %}
```python
def hist_chart(df):
    plt.hist(df['distance'], bins=25)
    plt.xlabel('Distance')
    plt.ylabel('Count')
    plt.title('Count of Throwers at Each Distance')
```
{% endtab %}

{% tab histogram Data %}
Fake data for this example:  
![Bar Chart](../static/visualizations%20tab/bar%20charts/distance_data.jpg)
{% endtab %}

{% tab histogram Comments %}
* `bins=25` sets bar count, grouping data into intervals.  
* Seaborn’s `sns.set_style('whitegrid')` adds a background grid.
{% endtab %}

{% endtabs %}

## Seaborn Colorful Bar Chart
Seaborn enhances bar charts with colors and backgrounds.

{% tabs sns_colorful %}

{% tab sns_colorful Image %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/seaborn_bars.png)
{% endtab %}

{% tab sns_colorful Code %}
```python
def sorted_bars(df):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    sns.set_style("darkgrid")
    df = df.sort_values(by='a_value', ascending=False)
    bar = sns.barplot(data=df, x='state', y='a_value', ax=ax)
    plt.xticks(rotation=60)
    plt.title('Best "A Value" by State')
    plt.ylabel('A Value')
    plt.xlabel('')
```
{% endtab %}

{% tab sns_colorful Data %}
First 11 rows of unsorted original data:  
![Bar Chart](../static/visualizations%20tab/bar%20charts/sorted_data.jpg)
{% endtab %}

{% tab sns_colorful Comments %}
* `sns.set_style("darkgrid")` sets the background.  
* Colors vary by default; set `colormap='xxx'` for options ([colormaps reference](https://matplotlib.org/2.0.2/examples/color/colormaps_reference.html)).  
* One value per state here.
{% endtab %}

{% endtabs %}

## Seaborn Statistical Bar Chart
Seaborn handles multiple values per category, showing statistics.

{% tabs sns_stats %}

{% tab sns_stats Image %}
![Bar Chart](../static/visualizations%20tab/bar%20charts/sns_bar_stats.jpg)
{% endtab %}

{% tab sns_stats Code %}
```python
def sns_bar_stats(df):
    sns.barplot(data=df, x='gender', y='distance', ci='sd')
    plt.ylabel('Distance')
    plt.xlabel('')
    plt.title('Avg with St Deviation of Distance by Gender')
```
{% endtab %}

{% tab sns_stats Data %}
Fake data for this example:  
![Bar Chart](../static/visualizations%20tab/bar%20charts/distance_data.jpg)
{% endtab %}

{% tab sns_stats Comments %}
* Line shows standard deviation (`ci='sd'`) for variance insight.  
* `ci` options (deprecated for `errorbar` in newer Seaborn):  
  * `None`: No error bar.  
  * Number (0-100): Confidence interval.  
  * `'sd'`: Standard deviation.  
* Default confidence interval uses [bootstrapping](https://www.thoughtco.com/what-is-bootstrapping-in-statistics-3126172) to reflect sample uncertainty.
{% endtab %}

{% endtabs %}
