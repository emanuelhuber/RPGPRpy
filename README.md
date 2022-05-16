# RPGPRpy
Python wrapper for RGPR (ground-penetrating radar visualisation &amp; processing).

*** 
**Work in progress!**
***

You find RGPR an interesting package for GPR (ground-penetrating radar) data processing but you do not like the R language... `RGPRpy` may be an alternative for you...


```py
# -*- coding: utf-8 -*-
"""
Created on Thu May 12 20:47:05 2022

https://github.com/davidthaler/Python-wrapper-for-R-Forecast/blob/master/rforecast/converters.py
https://github.com/theislab/anndata2ri/tree/master/anndata2ri

@author: Hubere
"""

import numpy as np
import matplotlib.pyplot as plt

import rpy2
print(rpy2.__version__)


from rpy2.robjects.packages import importr
# import R's "base" package
base = importr('base')

# import R's utility package
devtools = importr('devtools')

devtools.install_github("emanuelhuber/RGPR")

rgpr = importr('RGPR')

x = rgpr.readGPR("C:/Users/Hubere/Documents/GPR/BUF2____0001_1.rd3")

rgpr.plot_GPR(x)

# x = R object of type S4
type(x)

# class
tuple(x.rclass)

dir(x)

# Slots
tuple(x.slotnames())


# The attributes can also be accessed through the rpy2 property slots. slots is a mapping between attributes names (keys) and their associated R object (values). It can be used as Python dict:

# print keys
print(tuple(x.slots.keys()))

# fetch `phenoData`
xdat = x.slots['data']

tuple(xdat.rclass)

# Mapping S4 classes to Python classes
from rpy2.robjects.methods import RS4
class GPR(RS4):
    pass

x_myclass = GPR(x)


def rpy2py_s4(obj):
    if 'GPR' in obj.rclass:
        res = GPR(obj)
    else:
        res = robj
    return res

# try it
rpy2py_s4(x)


# The conversion system can also be made aware our new class by customizing the handling of S4 objects.

# A simple implementation is a factory function that will conditionally wrap the object in our Python class ExpressionSet:
from rpy2.robjects import default_converter
from rpy2.robjects.conversion import Converter, localconverter

my_converter = Converter('ExpressionSet-aware converter',
                         template=default_converter)

from rpy2.rinterface import SexpS4
my_converter.rpy2py.register(SexpS4, rpy2py_s4)



from multipledispatch import dispatch
from functools import partial

my_namespace = dict()
dispatch = partial(dispatch, namespace=my_namespace)

@dispatch(GPR)
def gainAGC(x, 
            w = 10, 
            p = 2, 
            r = 0.5, 
            track = False):
    res = rgpr.gainAGC(x,
                       w = w,
                       p = p,
                       r = r,
                       track = track)
    return res

res = gainAGC(x_myclass)

xdata = res.slots['data']
pydata = np.array(xdata)
plt.plot(pydata[:,10])

plt.imshow(pydata, interpolation='nearest', aspect = 'auto')
plt.show()


````
