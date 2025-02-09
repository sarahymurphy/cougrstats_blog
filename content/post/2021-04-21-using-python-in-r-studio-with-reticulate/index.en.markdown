---
title: Using Python in R Studio with Reticulate
author: Sarah Murphy
date: '2021-04-21'
slug: []
categories:
  - Package Introductions
tags:
  - R
  - Python
  - reticulate
description: Article description.
featured: yes
toc: no
featureImage: /images/reticulate_2021041_large.png
thumbnail: /images/reticulate_2021041_thumb.png
shareImage: /images/reticulate_2021041_thumb.png
codeMaxLines: 10
codeLineNumbers: no
figurePositionShow: yes
---

_By Sarah Murphy_

[Download all code used below from the GitHub repository](https://github.com/sarahymurphy/r-reticulate-tutorial)!

There are four ways to use Python code in your R workflow:

- Python code blocks in R Markdown
- Interactively in the console
- Importing Python packages and using the commands within your R scripts
- Importing Python scripts and using user-defined functions within your R scripts

All of these require reticulate.

## Reticulate
[Reticulate](https://rstudio.github.io/reticulate/) is a library that allows you to open a Python environment within R. You can also load Python packages and use them within your R script using a mix of Python and R syntax. All data types will be converted to their equivalent type when being handed off between Python and R.

### Installation

First, install the reticulate package: 

```r
install.packages("reticulate")
```

When you install reticulate you are also installing [Miniconda](https://docs.conda.io/en/latest/miniconda.html), a lightweight package manager for Python. This will create a new Python environment on your machine called `r-reticulate`. You do not need to use this environment but I will be using it for the rest of this post. [More information about Python environments can be found here](https://docs.python.org/3/tutorial/venv.html).

## Python Code Blocks in R Markdown

[R Markdown](https://rmarkdown.rstudio.com/) can contain both Python and R code blocks. When in an RMarkdown document you can either manually create a code block or click on the `insert` dropdown list in R Studio and select Python. Once you have a code block you can code using typical Python syntax. Python code blocks will run as Python code when you knit your document. 

![](https://github.com/sarahymurphy/r-reticulate-tutorial/blob/main/blogpost/img/python-codeblock.png?raw=true)

```
## 6
```

## Working with Python Interactively

We can use Python interactively within the console in R studio. To do this, we will use the `repl_python()` command. This opens a Python prompt.


```r
# Load library
library(reticulate)

# Open an interactive Python environment
# Type this in the console to open an interactive environment
repl_python()

# We can do some simple Python commands
python_variable = 4
print(python_variable)

# To edit the interactive environment
exit
```

You'll notice that when you're using Python `>>>` is displayed in the console. When you're using R, you should see `>`. 

### Changing Python environments
Using `repl_python()` will open the `r-reticulate` Python environment by default. If you've used Python and have a environment with packages loaded that you'd like to use, you can load that using the following commands.


```r
# I can see all my available Python versions and environments
reticulate::py_config()

# Load the Python installation you want to use
use_python("/Users/smurphy/opt/anaconda3/envs/r-reticulate/bin/python")

#  If you have a named environment you want to use you can load it by name
use_virtualenv("r-reticulate")
```

### Installing Python packages
Next, we need to install some Python packages:


```r
conda_install("r-reticulate", "cartopy", forge = TRUE)
conda_install("r-reticulate", "matplotlib")
conda_install("r-reticulate", "xarray")
```

The `conda_install` function uses Miniconda to install Python packages. We specify `"r-reticulate"` because that is the Python environment we want to install the packages into. We are going to install three different  packages here, [cartopy](https://scitools.org.uk/cartopy/docs/latest/) (for making maps), [matplotlib](https://matplotlib.org/) (a common plotting package), and [xarray](http://xarray.pydata.org/) (to import data).

## Using Python Commands within R Scripts

We can now load our newly installed Python libraries in R. Open a new R script and import these packages using the `import` command. We are assigning these to some common nicknames for these packages so they are easier to reference later on.


```r
library(reticulate)
```

```
## Warning: package 'reticulate' was built under R version 4.0.5
```

```r
plt <- import('matplotlib.pyplot')
ccrs <- import('cartopy.crs')
xr <- import('xarray')
feature <- import('cartopy.feature')
```

Xarray comes with some tutorial datasets. We are going to use the air temperature tutorial dataset here. To import it, we will use the xarray `open_dataset` command. We can now use all the usual Python commands, substituting `.` for `$`.


```r
air_temperature <- xr$tutorial$open_dataset("air_temperature.nc")

air_temperature
```

```
## <xarray.Dataset>
## Dimensions:  (lat: 25, lon: 53, time: 2920)
## Coordinates:
##   * lat      (lat) float32 75.0 72.5 70.0 67.5 65.0 ... 25.0 22.5 20.0 17.5 15.0
##   * lon      (lon) float32 200.0 202.5 205.0 207.5 ... 322.5 325.0 327.5 330.0
##   * time     (time) datetime64[ns] 2013-01-01 ... 2014-12-31T18:00:00
## Data variables:
##     air      (time, lat, lon) float32 ...
## Attributes:
##     Conventions:  COARDS
##     title:        4x daily NMC reanalysis (1948)
##     description:  Data is from NMC initialized reanalysis\n(4x/day).  These a...
##     platform:     Model
##     references:   http://www.esrl.noaa.gov/psd/data/gridded/data.ncep.reanaly...
```

Next, we will create our figure. This is a map of air temperature over the United States for any day in 2013 specified in the plotting command. 

Python users define lists with square brackets. However, the equivalent R type to this is a multi-element vector, so we must define it as such. One example of where this is used is in the first line in the following code block. In Python, we would specify the size of a figure using `fig = plt.figure(figsize = [15, 5])`, but when we use this command in R with reticulate we must use types that R recognizes, so we replace the `=` with `<-`, `.` with `$`, and `[]` with `c()` giving us `fig <- plt$figure(figsize = c(15, 5))`. 

The Python equivalent of the following R code can be seen in the next section about sourcing Python scripts. 


```r
# Creating the figure
fig <- plt$figure(figsize = c(15, 5))

# Defining the axes projection
ax <- plt$axes(projection = ccrs$PlateCarree())

# Setting the latitude and longitude boundaries
ax$set_extent(c(-125, -66.5, 20, 50))

# Adding coastlines
ax$coastlines()

# Adding state boundaries
ax$add_feature(feature$STATES);

# Drawing the latitude and longitude
ax$gridlines(draw_labels = TRUE);

# Plotting air temperature
plot <- air_temperature$air$sel(time='2013-04-14 00:00:00', method = 'nearest')$plot(ax = ax, transform = ccrs$PlateCarree())

# Giving our figure a title
ax$set_title('Air Temperature, 14 April 2013');

plt$show()
```


```
## <cartopy.mpl.feature_artist.FeatureArtist object at 0x00000000316F55F8>
```

```
## <cartopy.mpl.gridliner.Gridliner object at 0x00000000316F5588>
```

```
## Text(0.5, 1.0, 'Air Temperature\n2013-04-14 00:00:00')
```

```
## <cartopy.mpl.feature_artist.FeatureArtist object at 0x0000000031726F28>
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-2-1.png" width="1440" />

## Sourcing Python Scripts

We can easily import a Python script and use user-defined functions from it using `source_python`. The Python script below creates a function that does the same thing as the above R script. This function accepts a date string and plots the corresponding map.


```python
import xarray as xr
import matplotlib.pyplot as plt
import cartopy
import cartopy.crs as ccrs


def PlotAirTemp(usertime):
    """ Plot air temperature
    Args:
        username (str): "yyyy-mm-dd hh:mm:ss" 
                (for example '2013-04-14 00:00:00')
    """
    air_temperature = xr.tutorial.open_dataset("air_temperature.nc")
    fig = plt.figure(figsize = (15,5))
    ax = plt.axes(projection = ccrs.PlateCarree())
    ax.set_extent([-125, -66.5, 20, 50])
    ax.coastlines()
    ax.gridlines(draw_labels=True)
    plot = air_temperature.air.sel(time=usertime).plot(ax = ax, transform = ccrs.PlateCarree())
    ax.set_title('Air Temperature\n' + usertime)
    ax.add_feature(cartopy.feature.STATES)
    plt.show()
```

I have named the above file `Python_PlotAirTemp.py`. Once we source this file, we can use the function. Our function is called `PlotAirTemp`.


```r
library(reticulate)

# Source the Python file to import the functions
source_python('../Python_PlotAirTemp.py')

# Call the function and give it the appropriate arguments
PlotAirTemp('2013-04-14 00:00:00')
```




```
## <cartopy.mpl.feature_artist.FeatureArtist object at 0x0000000032882E10>
```

```
## <cartopy.mpl.gridliner.Gridliner object at 0x0000000032882DA0>
```

```
## Text(0.5, 1.0, 'Air Temperature\n2013-04-14 00:00:00')
```

```
## <cartopy.mpl.feature_artist.FeatureArtist object at 0x000000003287BB38>
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-2-1.png" width="1440" />

## Sources & Resources

- [A very clear YouTube tutorial using reticulate](https://www.youtube.com/watch?v=qATvD6kQ47s)
- [RStudio Blog - reticulate: R interface to Python](https://blog.rstudio.com/2018/03/26/reticulate-r-interface-to-python/)
- [RStudio- reticulate](https://rstudio.github.io/reticulate/)
- [R Markdown Python Engine](https://rstudio.github.io/reticulate/articles/r_markdown.html?_ga=2.8554995.750395877.1618538081-1626937029.1615660561)
- Python example based on code from [this xarray example](http://xarray.pydata.org/en/stable/examples/ERA5-GRIB-example.html) and [the xarray plotting reference](http://xarray.pydata.org/en/stable/plotting.html)

