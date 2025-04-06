---
layout: default_dl
title: Geospatial Plots
parent: Visualizations
nav_order: 5
---

# Geospatial Plots

Geospatial plots are great at:  
1. Showing how an **area** is impacted relative to other **areas**.  
2. Revealing which area is impacted.  
3. Illustrating a pattern in the location of something.

Geospatial plots are NOT good at:  
1. Showing **how** one value correlates to another. (Use a line plot for this)  
2. Emphasizing the amount of difference between one area and another.

## Contiguous States Only
Let’s start by showing the benefits of displaying only the contiguous United States. Imagine we’re trying to illustrate which states have the most electoral college votes, perhaps emphasizing that California has the most by a significant margin.

{% tabs contiguous_states %}

{% tab contiguous_states Image %}
On the left, we see a common student approach—showing all 50 states. On the right, we have a much-improved image displaying only the 48 contiguous states.  
![Election Image](../static/visualizations%20tab/geo_demo_clip.jpg)
{% endtab %}

{% tab contiguous_states Code %}
```python
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(15, 5))

# Left Side: Plot the entire United States
gdf.plot(ax=ax1, column='Electoral Votes')

# Right Side: Plot the contiguous US with a legend
gdf.plot(ax=ax2, column='Electoral Votes', legend=True)
ax2.axis('off')  # remove the box and the x,y ticks

# Set the extent of the axes to cut off Alaska and Hawaii
ax2.set_xlim(-130, -65)
ax2.set_ylim(24, 50)
```
{% endtab %}

{% tab contiguous_states Data %}
This data is a merge of two sources: `electoral_college.csv` and a ShapeFile of the United States. Note that in the code, we read column values like `"707,558"` (a string with a comma as the thousands separator) using the `thousands` parameter in `read_csv`. We also removed a duplicate column with `drop`.  
![Election Data](../static/visualizations%20tab/geo_elect_data.jpg)  
```python
electoral = pd.read_csv('electoral_college.csv', thousands=',')
gdf = us_states.merge(electoral, left_on='NAME', right_on='State', how='left')
gdf = gdf.drop(columns='State')  # this is a duplicate of 'NAME'
```
{% endtab %}

{% tab contiguous_states Comments %}
In this example, we present two geospatial plots in one figure with three key differences:  
1. The box and ticks are hidden.  
2. Alaska and Hawaii are clipped off.  
3. A legend is included.  

By excluding Alaska and Hawaii, we focus more on the contiguous states. The legend aids understanding, and removing the box with longitude/latitude ticks eliminates unnecessary distraction, as lat/long isn’t our focus. In the right chart, California’s size and bright yellow color highlight its significant impact.  

You can hide these states in two ways:  
- Filter them out of the GeoDataFrame: `gdf = gdf[(gdf['NAME'] != 'Alaska') & (gdf['NAME'] != 'Hawaii')]`.  
- Set axis limits: `ax2.set_xlim(-130, -65)`.
{% endtab %}

{% endtabs %}

## Inset Alaska & Hawaii
Excluding two states entirely isn’t ideal. We’d prefer to show Alaska and Hawaii as insets, like many maps do. This can be done, but it gets complex quickly. The inset is a separate plot with its own arguments, and adding legends adjusts figure size and inset placement. This code is included for educational purposes, but I recommend the simpler method below.

{: .note-title }
> Inset Image and Complicated Code
>
> ![Election Image](../static/visualizations%20tab/geo_inset_no_borders.jpg)  
> ```python
> def draw_us_inset(gdf, **args):
>     # Create a new figure and axes for the main plot
>     fig, ax = plt.subplots(1, figsize=(12, 5))
>
>     # Plot the entire United States
>     gdf.plot(ax=ax, **args)
>
>     # Set the extent of the main plot to cut off Alaska & Hawaii
>     ax.set_xlim(-130, -65)
>     ax.set_ylim(23, 50)
>
>     # Helper method to plot both insets without a border, no legend, correct vmin/vmax
>     def inset(position, name, xlim=None):
>         '''
>         position = a list: [x, y, width, height] in % of figure size
>         '''
>         ax = fig.add_axes(position)
>         state = gdf[gdf['NAME'] == name]
>         args_no_legend = args
>         args_no_legend['legend'] = False
>         # Set vmin/vmax to match the main plot for consistent colors
>         if 'column' in args:
>             args_no_legend['vmin'] = gdf[args['column']].min()
>             args_no_legend['vmax'] = gdf[args['column']].max()
>         state.plot(ax=ax, **args_no_legend)
>         # Remove box around inset in 1-line comprehension
>         [ax.spines[side].set_visible(False) for side in ['top', 'bottom', 'left', 'right']]
>         # Clip inset if needed
>         if xlim:
>             ax.set_xlim(xlim)
>         ax.set_xticks([])
>         ax.set_yticks([])
>
>     inset([0.20, 0.20, 0.20, 0.20], 'Alaska', xlim=(-180, -130))
>     inset([0.37, 0.19, 0.05, 0.15], 'Hawaii')
>
> draw_us_inset(gdf, column='Electoral Votes')
> ```

## Modified Geometry
Plotting the full United States becomes simpler if the GeoDataFrame’s geometry is adjusted to place Alaska and Hawaii in suitable inset positions. Drawbacks include:  
- Geospatial `sjoin`s may get geographically confused.  
- Pacific Ocean features might be obscured by relocated states.

{% tabs modified_geometry %}

{% tab modified_geometry Image %}
![Election Image](../static/visualizations%20tab/geo_elect.jpg)
{% endtab %}

{% tab modified_geometry Code %}
```python
from shapely.affinity import translate
from shapely.affinity import scale
from shapely.geometry import MultiPolygon

def modify_geometry(gdf, name_col='NAME'):
    '''
    Changes, in place, the geometry of Alaska & Hawaii in the GeoDataFrame to be "inset".
    name_col is the column containing the state names.
    '''
    def scale_translate(state_name, ratio=1.0, offset=(0, 0)):
        # Get geometry and save index using walrus operator
        multi_poly = gdf.loc[(index := gdf[gdf[name_col] == state_name].index[0]), 'geometry']
        # Scale each polygon in the state's MultiPolygon
        multi_poly = MultiPolygon([scale(poly, ratio, ratio) for poly in multi_poly])
        # Move the state by offset values
        multi_poly = translate(multi_poly, xoff=offset[0], yoff=offset[1])
        # Set value using 'at' method since 'loc' won't work
        gdf.at[index, 'geometry'] = multi_poly
    
    scale_translate('Alaska', ratio=0.35, offset=(26, -35))
    scale_translate('Hawaii', offset=(45, 5))

def plot_votes(gdf):
    fig, ax = plt.subplots(1, figsize=(10, 5))
    # Plot the entire United States
    gdf.plot(ax=ax, column='Electoral Votes', legend=True)
    ax.axis('off')  # Remove box and ticks
    # Set extent to cut off Alaska’s long tail
    ax.set_xlim(-130, -65)
    ax.set_ylim(24, 50)
    plt.title('Electoral College Votes by State')

# Call methods to generate plot with modified geometry
modify_geometry(gdf)
plot_votes(gdf)
```
{% endtab %}

{% tab modified_geometry Data %}
This data merges `electoral_college.csv` and a U.S. ShapeFile. We handled comma-separated values like `"707,558"` with the `thousands` parameter in `read_csv` and dropped a duplicate column.  
![Election Data](../static/visualizations%20tab/geo_elect_data.jpg)  
```python
electoral = pd.read_csv('electoral_college.csv', thousands=',')
gdf = us_states.merge(electoral, left_on='NAME', right_on='State', how='left')
gdf = gdf.drop(columns='State')  # Duplicate of 'NAME'
```
{% endtab %}

{% tab modified_geometry Comments %}
With modified geometry, plotting is straightforward and customizable. Notable points:  
- We use the walrus operator (`:=`) to assign a sub-expression within a larger expression, enhancing code conciseness. Two lines could achieve the same, but this earns _bragging rights_!  
- We use the `at` method instead of `loc` to modify the GeoDataFrame, avoiding a "ValueError: Must have equal len keys and value when setting with an iterable".
{% endtab %}

{% endtabs %}

## Annotated with Table Inset
The previous plot struggles to distinguish close values (e.g., Oregon vs. Nevada) due to similar colors. We’ll annotate each state with its vote count and add a table inset for smaller states.

{% tabs annotated_table %}

{% tab annotated_table Image %}
![Election Image](../static/visualizations%20tab/geo_elect_annotated.jpg)
{% endtab %}

{% tab annotated_table Code %}
```python
def annotate_states(gdf, ax, value_col, not_states=None, name_col='NAME'):
    # Annotate states with values from value_col, excluding not_states
    gdf_states = gdf if not_states is None else gdf[~gdf[name_col].isin(not_states)]
    for _, row in gdf_states.iterrows():
        x, y = row.geometry.centroid.x, row.geometry.centroid.y
        ax.annotate("{:.0f}".format(row[value_col]), (x, y), xytext=(-10, 0), textcoords='offset points')

def add_table_inset(fig, data):
    # Add table at [x, y, width, height] as % of figure
    ax_table = fig.add_axes([0.77, 0.35, 0.15, 0.4])
    ax_table.axis('off')
    table = ax_table.table(cellText=data.values, colLabels=data.columns, cellLoc='center', loc='center')
    table.auto_set_font_size(False)
    table.set_fontsize(10)
    table.scale(1, 1.5)

def plot_annotated_votes(gdf):
    fig, ax = plt.subplots(1, figsize=(10, 5))
    # Plot with 'cool' colormap for text visibility
    gdf.plot(ax=ax, column='Electoral Votes', legend=True, cmap='cool', 
             legend_kwds={'orientation': 'horizontal'}, edgecolor='lightgray')
    ax.axis('off')
    ax.set_xlim(-130, -65)
    ax.set_ylim(24, 50)
    plt.title('Electoral College Votes by State')
    
    small = ['District of Columbia', 'Rhode Island', 'New Jersey', 'Delaware', 'Vermont', 'New Hampshire', 'Connecticut']
    annotate_states(gdf, ax, 'Electoral Votes', not_states=small)
    
    # Create table data for small states with abbreviations
    df = gdf[gdf['NAME'].isin(small)].sort_values(by='NAME')
    d = {'State': ['CN', 'DE', 'DC', 'NH', 'NJ', 'RI', 'VT'],
         'Votes': list(df['Electoral Votes'].apply(lambda n: int(n)))}
    add_table_inset(fig, pd.DataFrame(data=d))

# Apply modified geometry and plot
modify_geometry(gdf)
plot_annotated_votes(gdf)
```
{% endtab %}

{% tab annotated_table Data %}
Same as above: merged `electoral_college.csv` and U.S. ShapeFile, with comma handling and column cleanup.  
![Election Data](../static/visualizations%20tab/geo_elect_data.jpg)  
```python
electoral = pd.read_csv('electoral_college.csv', thousands=',')
gdf = us_states.merge(electoral, left_on='NAME', right_on='State', how='left')
gdf = gdf.drop(columns='State')  # Duplicate of 'NAME'
```
{% endtab %}

{% tab annotated_table Comments %}
We made the legend horizontal using `legend_kwds`, just because we can. Key issues addressed:  

| Problem | Fix |
|---------|-----|
| Black text blends into the default colormap | Use 'cool' colormap |
| Small states lack space for text | Skip annotating small states |
| Small states’ values are lost | Add an inset table |
| Floating-point values in table | Convert to integers with `apply` and `lambda` |

We could’ve used 'CENSUSAREA' to filter small states, but explicit listing works here.
{% endtab %}

{% endtabs %}

## Finding Obvious Correlations
Do population or state size (square miles) correlate with electoral votes? Geospatial plots offer limited insight; scatter plots are better.

{% tabs correlations_obvious %}

{% tab correlations_obvious Image %}
Top-left: Electoral Votes by state. A population plot would look nearly identical, suggesting correlation. Top-right: State size in square miles.  
![Election Image](../static/visualizations%20tab/geo_correlation.jpg)
{% endtab %}

{% tab correlations_obvious Code %}
```python
def plot_correlation(gdf):
    fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(10, 5))
    gdf.plot(ax=ax1, column='Electoral Votes', legend=True)
    gdf.plot(ax=ax2, column='CENSUSAREA', legend=True)
    for ax in (ax1, ax2):
        ax.set_xlim(-130, -65)
        ax.set_ylim(24, 50)
    ax1.set_title('Electoral Votes')
    ax2.set_title('Square Miles')
    gdf.plot(ax=ax4, kind='scatter', y='Electoral Votes', x='CENSUSAREA', cmap='viridis')
    gdf.plot(ax=ax3, kind='scatter', y='Electoral Votes', x='2023 Population')
    ax4.set_title('Area vs Votes')
    ax3.set_title('Population vs Votes')
    for ax in (ax3, ax4):
        ax.grid(linestyle="--", linewidth=0.5, color='.25', zorder=-10)
```
{% endtab %}

{% tab correlations_obvious Data %}
Same as above.  
![Election Data](../static/visualizations%20tab/geo_elect_data.jpg)
{% endtab %}

{% tab correlations_obvious Comments %}
A population geospatial plot would mirror the votes plot, indicating strong correlation (votes are population-based). We skipped it for a scatter plot. The square miles plot suggests no obvious correlation, confirmed by scatter plots. Combining all four plots clarifies: population strongly correlates with votes, while area does not.  

{: .note-title }
> Using Statistics
>
> Other sections show how Python libraries calculate metrics like Coefficient of Determination ($R^2$). Here, $R^2$ for Votes vs. Area is 0.019 (very low), vs. 0.991 for Votes vs. Population (very high).
{% endtab %}

{% endtabs %}

## Correlations and Coefficient of Determination
We create three values (A, B, C) per state with strong correlations. Geospatial plots obscure the "how," but scatter and line plots clarify it. See tabs for details.

{% tabs correlations_coeff %}

{% tab correlations_coeff Image %}
![Geospatial & Linear Plots](../static/visualizations%20tab/geo_line_correlations.jpg)
{% endtab %}

{% tab correlations_coeff Code %}
```python
from scipy import stats

def plot_correlations(gdf):
    gdf.sort_values(by='A')
    res_b = stats.linregress(gdf['A'], gdf['B'])
    res_c = stats.linregress(gdf['A'], gdf['C'])
    fig, ((ax1, ax2), (ax3, ax4), (ax5, ax6), (ax7, ax8)) = plt.subplots(4, 2, figsize=(12, 15))
    plt.subplots_adjust(hspace=0.3)
    max_value = max(gdf['A'].max(), gdf['B'].max(), gdf['C'].max())
    gdf.plot(ax=ax1, column='A', legend=True)
    gdf.plot(ax=ax2, column='B', legend=True, vmin=0, vmax=max_value)
    gdf.plot(ax=ax3, column='C', legend=True)
    gdf.plot(ax=ax4, column='A', legend=True, vmin=0, vmax=max_value)
    gdf.plot(ax=ax5, column='C', legend=True, vmin=0, vmax=max_value)
    axes = (ax1, ax2, ax3, ax4, ax5)
    for ax in axes:
        ax.set_xlim(-130, -65)
        ax.set_ylim(24, 50)
    for ax, s in zip(axes, ['Value A', 'Value B', 'Value C', 'Value A (on B scale)', 'Value C (on B scale)']):
        ax.set_title(s)
    for ax in (ax6, ax8):
        gdf.plot(ax=ax, kind='scatter', x='A', y='B', color='blue', label='B Value')
    for ax in (ax7, ax8):
        gdf.plot(ax=ax, kind='scatter', x='A', y='C', marker='s', color='green', label='C Value')
    line = gdf[['A', 'B', 'C']].copy()
    line['YB'] = res_b.intercept + res_b.slope * line['A']
    line['YC'] = res_c.intercept + res_c.slope * line['A']
    for ax in (ax6, ax8):
        line.plot(ax=ax, kind='line', x='A', y='YB', color='red', label='B fitted line')
    for ax in (ax7, ax8):
        line.plot(ax=ax, kind='line', x='A', y='YC', color='black', label='C fitted line')
    ax6.text(70, 20, f'R2={res_b.rvalue**2:.3f}')
    ax7.text(70, 20, f'R2={res_c.rvalue**2:.3f}')
    ax8.text(70, 20, f'B R2={res_b.rvalue**2:.3f}\nC R2={res_c.rvalue**2:.3f}')
    ax6.set_title('Value B vs Value A')
    ax7.set_title('Value C vs Value A')
    ax8.set_title('B & C vs Value A')
```
{% endtab %}

{% tab correlations_coeff Data %}
Contrived values for A, B, and C (first 12 rows shown).  
![Election Data](../static/visualizations%20tab/geo_abc_data.jpg)
{% endtab %}

{% tab correlations_coeff Plot Comments %}
Geospatial plots hint at correlations, but details are unclear. **Value A** and **Value C** seem similar, yet scaling to B’s range reveals differences. The relationship with **Value B** is vague. Scatter plots with best-fit lines clarify linear relationships, and $R^2$ values (0.736 for B, 0.948 for C) quantify strength. The final combined plot is most effective for showing correlations with **Value A**.
{% endtab %}

{% tab correlations_coeff Code Comments %}
We use `stats.linregress` for line fitting, requiring sorted x-axis data. `seaborn.regplot` could draw lines but lacks $R^2$. We position $R^2$ text with `ax.text`, formatted via f-strings (e.g., `f'R2={value:.3f}'`). Matching color scales uses `vmin` and `vmax`. Vertical spacing is adjusted with `plt.subplots_adjust(hspace=0.3)`; `GridSpec` could refine this but is omitted for simplicity.
{% endtab %}

{% endtabs %}

## Cartopy
`Cartopy` creates professional terrain plots, ideal for showing locations on a map.  
**Advantages:**  
- Downloads data automatically, no manual shapefiles needed.  
- High-quality visuals.  
**Disadvantages:**  
- Incompatible with Replit.  
- Requires understanding of coordinate/projection systems.

{% tabs cartopy %}

{% tab cartopy Image %}
Shows a flat Plate Carrée line and a curved Geodetic line (airplane route) between Seattle and New York.  
![Election Image](../static/visualizations%20tab/geo_cartopy.jpg)
{% endtab %}

{% tab cartopy Code %}
```python
import cartopy.crs as ccrs
import matplotlib.pyplot as plt
import cartopy.feature as cfeature

def cartopy_plot_of_us():
    fig, ax = plt.subplots(1, subplot_kw={'projection': ccrs.Robinson()}, figsize=(10, 4))
    ax.stock_img()
    ax.set_extent([-130, -60, 24, 50], crs=ccrs.PlateCarree())
    ny_lat, ny_lon = 40.73, -73.93
    sea_lat, sea_lon = 47.60, -122.33
    plt.plot([ny_lon, sea_lon], [ny_lat, sea_lat], color='blue', linewidth=2, marker='o', transform=ccrs.Geodetic())
    plt.plot([ny_lon, sea_lon], [ny_lat, sea_lat], color='gray', linestyle='--', transform=ccrs.PlateCarree())
    plt.text(ny_lon - 3, ny_lat - 1, 'New York', horizontalalignment='right', transform=ccrs.Geodetic())
    plt.text(sea_lon + 1, sea_lat - 2, 'Seattle', horizontalalignment='left', transform=ccrs.Geodetic())
    ax.add_feature(cfeature.COASTLINE)
    ax.add_feature(cfeature.STATES, linestyle=":")
    ax.add_feature(cfeature.BORDERS)
    ax.add_feature(cfeature.LAKES, alpha=0.7)
    ax.add_feature(cfeature.RIVERS)
```
{% endtab %}

{% tab cartopy Data %}
Data is downloaded by Cartopy; no external files needed.
{% endtab %}

{% tab cartopy Comments %}
Projections like `ccrs.PlateCarree()` are common. We add a textured background, plot lines with different projections, label cities, and include features like borders and rivers. Only city coordinates are required. Learning Cartopy takes time, so ensure it suits your needs before diving in.
{% endtab %}

{% endtabs %}