---
toc: true
layout: post
description: How to make an example in cartopy working again.
categories: [python, matplotlib, cartopy]
title: Set margins in Cartopy
---
# Make `set_margins` in `Cartopy` working again

## Problem

One of the Cartopy examples in section [More advanced mapping with cartopy and matplotlib](https://scitools.org.uk/cartopy/docs/v0.18/matplotlib/advanced_plotting.html#images) has not been working ever since v0.16. The margins specified in the code failed to show up in the plot. The code is as follows:

```python
import os
import matplotlib.pyplot as plt

from cartopy import config
import cartopy.crs as ccrs


fig = plt.figure(figsize=(8, 12))

# get the path of the file. It can be found in the repo data directory.
fname = os.path.join(config["repo_data_dir"],
                     'raster', 'sample', 'Miriam.A2012270.2050.2km.jpg'
                     )
img_extent = (-120.67660000000001, -106.32104523100001, 13.2301484511245, 30.766899999999502)
img = plt.imread(fname)

ax = plt.axes(projection=ccrs.PlateCarree())
plt.title('Hurricane Miriam from the Aqua/MODIS satellite\n'
          '2012 09/26/2012 20:50 UTC')

# set a margin around the data
ax.set_xmargin(0.05)
ax.set_ymargin(0.10)

# add the image. Because this image was a tif, the "origin" of the image is in the
# upper left corner
ax.imshow(img, origin='upper', extent=img_extent, transform=ccrs.PlateCarree())
ax.coastlines(resolution='50m', color='black', linewidth=1)

# mark a known place to help us geo-locate ourselves
ax.plot(-117.1625, 32.715, 'bo', markersize=7, transform=ccrs.Geodetic())
ax.text(-117, 33, 'San Diego', transform=ccrs.Geodetic())

plt.show()
```

---
## Solution

I first thought it was caused by Cartopy itself, but couldn't find any changes in Cartopy that can cause this. Then I started to check Matplotlib code, and found out that there were several internal variables in Matplotlib controlling the margin behavior. After some diagnosis, and based on the [Matplotlib documentation](https://matplotlib.org/3.4.0/gallery/subplots_axes_and_figures/axes_margins.html#on-the-stickiness-of-certain-plotting-methods), Matplotlib introduced a new control mechanism for the stickiness of plots (i.e., the margins), `use_sticky_edges`, which defaults to `True`, since version 2.0, at about the same time as Cartopy v0.15. If this is not desired (as in this case), it should be set as `False`. So the solution is to insert one line 
```python
ax.use_sticky_edges = False
```
before
```python
# set a margin around the data
ax.set_xmargin(0.05)
ax.set_ymargin(0.10)
```
The output is 

![](https://scitools.org.uk/cartopy/docs/v0.15/_images/advanced_plotting-3.png)

I have put a [PR](https://github.com/SciTools/cartopy/pull/1750) to Cartopy, which has been merged and should be included in the upcoming v0.19. 
