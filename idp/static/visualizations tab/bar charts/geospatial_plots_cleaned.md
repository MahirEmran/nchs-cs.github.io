---
layout: default_dl
title: Geospatial Plots
parent: Data Visualization
nav_order: 5
---

# Geospatial Plots

Geospatial plots are useful for:
1. Showing how an **area** is impacted relative to other **areas**
2. Revealing **which** area is impacted
3. Illustrating **location-based patterns**

They are **not** great for:
1. Showing **correlation** between variables (use scatter or line plots)
2. Emphasizing the **magnitude** of difference between regions

## Contiguous States Only

Let’s compare plots with all 50 states vs. just the contiguous 48 to emphasize differences more clearly—like California’s significantly higher electoral votes.

````{tab-set}
```{tab-item} Image
Left: All 50 states  
Right: 48 contiguous states only  
![election image](../_static/geo_demo_clip.jpg)
```

```{tab-item} Code
```python
fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(15, 5))
gdf.plot(ax=ax1, column='Electoral Votes')
gdf.plot(ax=ax2, column='Electoral Votes', legend=True)
ax2.axis('off')
ax2.set_xlim(-130, -65)
ax2.set_ylim(24, 50)
```

```{tab-item} Data
```python
electoral = pd.read_csv('electoral_college.csv', thousands=',')
gdf = us_states.merge(electoral, left_on='NAME', right_on='State', how='left')
gdf = gdf.drop(columns='State')
```
````

## Inset Alaska & Hawaii

Use inset plots to show Alaska and Hawaii in a better context.

````{tab-set}
```{tab-item} Image
![election image](../_static/geo_inset_no_borders.jpg)
```

```{tab-item} Code
```python
def draw_us_inset(gdf, **args):
    fig, ax = plt.subplots(1, figsize=(12, 5))
    gdf.plot(ax=ax, **args)
    ax.set_xlim(-130, -65)
    ax.set_ylim(23, 50)

    def inset(position, name, xlim=None):
        ax_inset = fig.add_axes(position)
        state = gdf[gdf['NAME'] == name]
        args_no_legend = args.copy()
        args_no_legend['legend'] = False
        if 'column' in args:
            args_no_legend['vmin'] = gdf[args['column']].min()
            args_no_legend['vmax'] = gdf[args['column']].max()
        state.plot(ax=ax_inset, **args_no_legend)
        [ax_inset.spines[side].set_visible(False) for side in ['top', 'bottom', 'left', 'right']]
        if xlim:
            ax_inset.set_xlim(xlim)
        ax_inset.set_xticks([])
        ax_inset.set_yticks([])

    inset([0.20, 0.20, 0.20, 0.20], 'Alaska', xlim=(-180, -130))
    inset([0.37, 0.19, 0.05, 0.15], 'Hawaii')

draw_us_inset(gdf, column='Electoral Votes')
```
````

## Modified Geometry

Easily reposition Alaska and Hawaii for better integrated plots.

````{tab-set}
```{tab-item} Image
![election image](../_static/geo_elect.jpg)
```

```{tab-item} Code
```python
from shapely.affinity import translate, scale
from shapely.geometry import MultiPolygon

def modify_geometry(gdf, name_col='NAME'):
    def scale_translate(state_name, ratio=1.0, offset=(0, 0)):
        idx = gdf[gdf[name_col] == state_name].index[0]
        multi_poly = gdf.at[idx, 'geometry']
        multi_poly = MultiPolygon([scale(poly, ratio, ratio) for poly in multi_poly])
        multi_poly = translate(multi_poly, xoff=offset[0], yoff=offset[1])
        gdf.at[idx, 'geometry'] = multi_poly

    scale_translate('Alaska', ratio=0.35, offset=(26, -35))
    scale_translate('Hawaii', offset=(45, 5))

def plot_votes(gdf):
    fig, ax = plt.subplots(1, figsize=(10, 5))
    gdf.plot(ax=ax, column='Electoral Votes', legend=True)
    ax.axis('off')
    ax.set_xlim(-130, -65)
    ax.set_ylim(24, 50)
    plt.title('Electoral College Votes by State')

modify_geometry(gdf)
plot_votes(gdf)
```

```{tab-item} Data
![election data](../_static/geo_elect_data.jpg)
```python
electoral = pd.read_csv('electoral_college.csv', thousands=',')
gdf = us_states.merge(electoral, left_on='NAME', right_on='State', how='left')
gdf = gdf.drop(columns='State')
```
````

> Let me know if you'd like the next sections too (Annotations, Correlations, Cartopy).