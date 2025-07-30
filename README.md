
# Hausdorff

[![Badge-Version]][Package] [![Badge-Downloads]][Package]

Implementation of the [Hausdorff Distance] algorithm presented in the  
paper '[*An Efficient Algorithm for Calculating the Exact Hausdorff Distance*][Paper]' <sup>[1]</sup>  
by [Abdel Aziz Taha] & [Allan Hanbury].

<br/>

**[1]** DOI: `10.1109/TPAMI.2015.2408351`

<br/>

## Installation

### Package

You can install the [PyPi Package][Package].

```bash
pip install hausdorff
```

### Manual

Or clone and set it up yourself.

```bash
python setup.py install
```

<br/>

## Usage

The main functions is: 

`hausdorff_distance(np.ndarray[:,:] X, np.ndarray[:,:] Y)`

Which computes the _Hausdorff distance_ between the rows of `X` and `Y` using the Euclidean distance as metric. It receives the optional argument `distance` (**string or callable**), which is the distance function used to compute the distance between the rows of `X` and `Y`. In case of string, it could be any of the following: `manhattan`, `euclidean` (default), `chebyshev` and `cosine`. In case of callable, it should be a numba decorated function (see example below).


__Note:__ The haversine distance is calculated assuming lat, lng coordinate ordering and assumes
 the first two coordinates of each point are latitude and longitude respectively.
 
<br/>

 ### Basic

```python
import numpy as np
from hausdorff import hausdorff_distance

# two random 2D arrays (second dimension must match)
np.random.seed(0)
X = np.random.random((1000,100))
Y = np.random.random((5000,100))

# Test computation of Hausdorff distance with different base distances
print(f"Hausdorff distance test: {hausdorff_distance(X, Y, distance='manhattan')}")
print(f"Hausdorff distance test: {hausdorff_distance(X, Y, distance='euclidean')}")
print(f"Hausdorff distance test: {hausdorff_distance(X, Y, distance='chebyshev')}")
print(f"Hausdorff distance test: {hausdorff_distance(X, Y, distance='cosine')}")

# For haversine, use 2D lat, lng coordinates
def rand_lat_lng(N):
    lats = np.random.uniform(-90, 90, N)
    lngs = np.random.uniform(-180, 180, N)
    return np.stack([lats, lngs], axis=-1)
        
X = rand_lat_lng(100)
Y = rand_lat_lng(250)
print("Hausdorff haversine test: {0}".format( hausdorff_distance(X, Y, distance="haversine") ))
```

<br/>

### Custom

The distance function is used to calculate the distances between the rows of the input 2-dimensional arrays . For optimal performance, this custom distance function should be decorated with `@numba` in [nopython mode](https://numba.pydata.org/numba-doc/latest/user/jit.html).

```python
import numba
import numpy as np
from math import sqrt
from hausdorff import hausdorff_distance

# two random 2D arrays (second dimension must match)
np.random.seed(0)
X = np.random.random((1000,100))
Y = np.random.random((5000,100))

# write your own crazy custom function here
# this function should take two 1-dimensional arrays as input
# and return a single float value as output.
@numba.jit(nopython=True, fastmath=True)
def custom_dist(array_x, array_y):
    n = array_x.shape[0]
    ret = 0.
    for i in range(n):
        ret += (array_x[i]-array_y[i])**2
    return sqrt(ret)

print(f"Hausdorff custom euclidean test: {hausdorff_distance(X, Y, distance=custom_dist)}")

# a real crazy custom function
@numba.jit(nopython=True, fastmath=True)
def custom_dist(array_x, array_y):
    n = array_x.shape[0]
    ret = 0.
    for i in range(n):
        ret += (array_x[i]-array_y[i])**3 / (array_x[i]**2 + array_y[i]**2 + 0.1)
    return ret

print(f"Hausdorff custom crazy test: {hausdorff_distance(X, Y, distance=custom_dist)}")
```

[Badge-Downloads]: http://img.shields.io/pypi/dm/hausdorff.svg?style=for-the-badge
[Badge-Version]: http://img.shields.io/pypi/v/hausdorff.svg?style=for-the-badge

[Hausdorff Distance]: https://en.wikipedia.org/wiki/Hausdorff_distance
[Abdel Aziz Taha]: https://ieeexplore.ieee.org/author/37085470224
[Allan Hanbury]: https://ieeexplore.ieee.org/author/37269249300
[Package]: https://pypi.org/project/hausdorff/
[Paper]: https://ieeexplore.ieee.org/document/7053955