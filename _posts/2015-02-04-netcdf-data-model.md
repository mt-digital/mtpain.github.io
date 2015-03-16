---
title: netCDF-(c and python) on OS X Yosemite
tags: software development data netcdf
layout: post
date: 2015-02-04
---

# Intro

This post will show you how to get started quickly writing .nc files from
the NetCDF-C interface on Mac OS X. The installation instructions will be
different for Linux, but how to compile and run a program will be the same.  
We'll also cover the abstract NetCDF data model.

# The basics: Install netCDF-4 and write/compile a program

First, I'll show how to install NetCDF4. By this I mean the C libraries and
header files needed to write and compile NetCDF-C code. These files are also
required for installing and using NetCDF-python.

Next, I'll show how to create a NetCDF `Dataset` object in python, and then
we'll get a little closer to the machine and walk through the same example in C. 

The goal in all this is to implement the [NetCDF Data
Model](http://www.unidata.ucar.edu/software/netcdf/docs/html_tutorial/netcdf_data_model.html),
the standard conceptualization of data and metadata all in one package, which
NetCDF refers to as a `Dataset`. Below we show the so-called "classic" model,
which is simpler and can be considered a minimum `Dataset`. 

![Classic Data
Model](http://www.unidata.ucar.edu/software/netcdf/docs/html_tutorial/nc-classic-uml.png
"NetCDF Classic Data Model")

Taken from [UNIDATA
site](http://www.unidata.ucar.edu/software/netcdf/docs/html_tutorial/nc-classic-uml.png).
The NetCDF Enhanced Data Model allows for rich user-defined types as well, and
I expect will serve us well.

### NetCDF4 Installation 

I had figure this out once before, but then referenced [Alejandro Soto's 
"Setting Up My Mac for Scientific Research"][as_setup] to recall. I also learned
a new tool for plotting netCDF's, which he explains, called [GRADS: grid
analysis and display system](http://www.iges.org/grads/).

But we'll focus only on installing and using netCDF. We'll be using the 
ever-useful Homebrew for installing.
If you don't have Homebrew, you should [get it; it's truly "the missing package manager for OSX"
](http://brew.sh/). Read up on how it works if you want, but usually it just
works or gives useful instructions if something goes wrong.

Finally, we'll actually install netCDF. 
First, tap the homebrew-science keg and install netcdf. 

{% highlight bash %}
$ brew tap homebrew/science
{% endhighlight %}

Next, install netcdf

{% highlight bash %}
$ brew install netcdf
{% endhighlight %}

Simple!

Finally, install the Python NetCDF bindings:

{% highlight bash %}
$ pip install netcdf4
{% endhighlight %}

There may be other dependencies, but pip will let you know about it.


## NetCDF-python

* The [reference documentation](http://unidata.github.io/netcdf4-python/)
is very good, a must-read for working with NetCDF in general and in Python.
* I just found a nice [IPython Notebook
Tutorial](http://nbviewer.ipython.org/github/cedadev/ceda_training_mod5/blob/master/03-Creating_NetCDF.ipynb)
that mirrors the [NetCDF-C tutorial
](http://www.unidata.ucar.edu/software/netcdf/docs/html_tutorial/index.html).

Not sure what I'll eventually use. For now I need to get something done pretty
fast with the progress I've made so far. That means going through some 
NetCDF-Python examples, though I'm stoked about the C implementations as well.

Why the NetCDF folks didn't just do a copy of the C example program of the 4D
creation of a simple dataset, I don't know, but they didn't. So here it is. See
the C version [pres_temp_4D_wr.c
example](http://www.unidata.ucar.edu/software/netcdf/docs/html_tutorial/pres__temp__4D__wr_8c.html)
at the UCAR NetCDF documentation page. 
If you have at least a little bit of C experience, I highly recommend going 
through the C example as well, it's very instructive and makes me appreciate a 
Python implementation all the more. 

In this example, we'll generate a Dataset
with four dimensions: time, "level", latitude, and longitude. There will be two
non-dimensional variables, pressure and temperature. We will create fake but
plausible values for both temperature and pressure, and latitude and longitude
as well. I call pressure and temperature "non-dimensional" variables, because
the dimensions need values themselves. Here it is:

{% highlight python %}
"""
File: pres_temp_4D_wr.py

An example of how to create a new NetCDF dataset based on the NetCDF-C
tutorial version in C, found at
http://www.unidata.ucar.edu/software/netcdf/docs/html_tutorial/pres__temp__4D__wr_8c.html

I also followed the NetCDF4-Python examples found at
http://unidata.github.io/netcdf4-python/
"""
import time
import numpy as np

from netCDF4 import Dataset

# All NetCDF exists with the expectation that it will be written to
# disk, so it's required to provide a file name when instantiating a
# Dataset (see http://unidata.github.io/netcdf4-python/netCDF4.Dataset-class.html)
file_name = "pres_temp_4D.nc"

# default is to overwrite existing file. Can create in-memory via diskless=True
dataset = Dataset(file_name, 'w', )

### Create our four dimensions ###
# work with a 6 x 12 grid
NLAT = 6
NLON = 12
# The 'None's mean that there is no limit to their number
# see http://unidata.github.io/netcdf4-python/netCDF4.Dataset-class.html#createDimension
time_dim = dataset.createDimension('time', None)
level_dim = dataset.createDimension('level', None)
lat_dim = dataset.createDimension('lat', NLAT)
lon_dim = dataset.createDimension('lon', NLON)

# Now we create variables. All four dimensions are variables, and we'll add
# two more non-dimensional variables.
# The second argument is the data type: 4-byte int, 8-byte float, etc.
time_var = dataset.createVariable('time', 'f8', ('time', ))
level_var = dataset.createVariable('level', 'i4', ('level', ))
lat_var = dataset.createVariable('lat', 'f4', ('lat', ))
lon_var = dataset.createVariable('lon', 'f4', ('lon', ))
# for non-dimensional variables, we must declare the dimensions they vary over
pres_var = dataset.createVariable('pres', 'f4',
                                  ('time', 'level', 'lat', 'lon'))
temp_var = dataset.createVariable('temp', 'f4',
                                  ('time', 'level', 'lat', 'lon'))

# now we set some attributes: units and other metadata
# Possibly confusingly, these are set using variable methods, not dimension
# Though this makes sense since we need metadata not only for dimensions
dataset.description = 'Sample NetCDF File ported from NetCDF-C tutorial'
dataset.history = 'Created ' + time.ctime(time.time())
dataset.source  = 'Rebirth Galaxy\'s tutorial'
lat_var.units = 'degrees north'
lon_var.units = 'degrees east'
level_var.units = 'meters'  # could be water table data, for example
temp_var.units = 'C'
time_var.units = 'hours since 2010-01-01 00:00:00'
time_var.calendar = 'gregorian'  # duh, (http://unidata.github.io/netcdf4-python/)

start_lat = 25.0
start_lon = -120.0
# lat_var[:] = np.arange([start_lat + 0.5*i for i in range(NLAT)])
lat_var[:] = np.array([start_lat + 0.5*i for i in range(NLAT)])
lon_var[:] = np.array([start_lon + 0.5*i for i in range(NLON)])
# not too worried about realistic times and levels for now
time_var[:] = np.array([0,1])
level_var[:] = np.array([0,1])

# Now let's create some data for each element of the temperature and pressure
# we will use the same 3D data for each time step.
temp_data = np.zeros((2, NLAT, NLON))
pres_data = np.zeros((2, NLAT, NLON))

start_temp = 9.0
start_pres = 900

i = 0
for lvl in range(len(level_var[:])):
    for lat in range(len(lat_var[:])):
        for lon in range(len(lon_var[:])):
            temp_data[lvl, lat, lon] = start_temp + i
            pres_data[lvl, lat, lon] = start_pres + i
            i += 1

full_temp = np.array([temp_data]*2)
full_pres = np.array([pres_data]*2)

temp_var[:] = full_temp
pres_var[:] = full_pres

# closing the Dataset saves the file
dataset.close()
{% endhighlight %}



## NetCDF-C

### Compile and run a sample program

I figure if I'd had more experience writing C code (I used [D](http://dlang.org)
at my last job and never really learned C/C++), I would not have had trouble
compiling a program. This is for those folks like me who are relatively new
working with C. 

For this part of the tutorial, head to the [NetCDF-C Tutorial](http://www.unidata.ucar.edu/software/netcdf/docs/html_tutorial/index.html) 
and code up your own version of a [simple
example that writes two variables to a .nc
file](http://www.unidata.ucar.edu/software/netcdf/docs/html_tutorial/simple__xy__wr_8c.html). 

I had gone through the whole example, happily writing away, and compiled like
so:

{% highlight bash %}
cc simple_xy_wr.c
{% endhighlight %}

but got this error:

{% highlight bash %}
Undefined symbols for architecture x86_64:
  "_nc_close", referenced from:
      _main in simple_xy_wr-1220be.o
  "_nc_create", referenced from:
      _main in simple_xy_wr-1220be.o

        ...

ld: symbol(s) not found for architecture x86_64
{% endhighlight %}

By adding `-v` to the output, I could see that the search paths for the `include
<..>` statement was good, because `netcdf.h` was in the first search path, 
`/usr/local/include`. So what was the problem? The problem was that CC couldn't
find the library of functions. It could find the prototypes in `netcdf.h` fine,
but it didn't know what were the actual functions. Easily fixed by letting `cc`
know the library to be by changing our compilation command to 

{% highlight bash %}
cc -lnetcdf simple_xy_wr.c
{% endhighlight %}

Then we can run `./a.out` and see the results. If you want the resulting
executable named something better, run 

{% highlight bash %}
cc -lnetcdf simple_xy_wr.c -o better_name
{% endhighlight %}

then run your freshly-compiled programming by exectuing `./better_name`.

[as_setup]: http://alejandrosoto.net/blog/2014/01/22/setting-up-my-mac-for-scientific-research/ "Setting Up My Mac for Scientific Research"
