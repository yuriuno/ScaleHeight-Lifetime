# scaleheight_lifetime

This scaleheight-lifetime module helps you estimate a prousible scale height and lifetime of stars.<br>
The code was written during the investigation of the OH/IR stars distribution in the Galaxy ([Uno et al. 2020]()).
## Requirements
The code was implemented on python3.7 and it uses the following python liblalies. 
- numpy
- matplotlib
- scipy
- pylab
## Setup
**package installation**

`pip install git+https://github.com/yuriuno/scaleheight-lifetime.git`

## Data analysis

### 1. Scale height
#### 1.1 Construct a simple Galactic disk model 
```
import numpy as np
import matplotlib.pyplot as plt
from pylab import rcParams
from mpl_toolkits.mplot3d import Axes3D
from scipy import stats
import scaleheight_lifeitme.scaleheight_lifetme as sl

sh            = sl.ScaleHeight()
NUMBER        = 3990
SCALE_LENGTH  = 22   # kpc *exponential disk model
SCALE_HEIGHT  = 0.15 # kpc *gaussian (simplified sech z^2 disk model)
DIST_FUNC     = {'scale_length':SCALE_LENGTH, 'scale_height':SCALE_HEIGHT}
KINE_DIST     = 'near' # 'near' or  'far' * luminosity function based on near/far kinematic distance    
DETECTION_LIM = 0.122 # Jy   *SPLASH detection limit is 0.065 * 3 sigma 
mxyz, mlb, mflux = sh.model(DIST_FUNC, NUMBER, KINE_DIST)
```

#### 1.2 Evaluate your model using K-S test

```
DATA   = np.load('./data/stat') # datatype: numpy strings
# DATA format: name, longitude, latitude, radial-velocity, expansion-velocity, velocity-uncertainity, flux-density, size, spot_num
REGION  = [{'l':[-28, -4], 'b':[-1.25, 1.25]},{'l':[4, 10], 'b':[-1.25,1.25]}, {'l':[14.35, 67.25], 'b':[-1.25,1.25]}] # specify the feild where you are evaluating
# dlb: np.array([l,b]) 
dlb     = np.array([[float(i) for i in DATA[1]],[float(i) for i in DATA[2]]])
p_value = sh.pvalue(dlb, mlb, mflux, DETECTION_LIM, REGION)
print(format(p_value,'.3f'))
>>> '0.531'
```

#### 1.3 Quick visualization of the constructed model.
```
# 3D(X,Y,Z) [kpc]
sh.visualize(mxyz, mflux)
```
<img width="500" alt="3dmodel" src="https://user-images.githubusercontent.com/49733387/101880194-da365a00-3bd5-11eb-902e-0e970a354806.png">

```
# 2D(l,b) [deg]
sh.visualize(mlb, mflux)
```
<img width="500" alt="2dmodel" src="https://user-images.githubusercontent.com/49733387/101880254-ee7a5700-3bd5-11eb-9b02-3741a968b6cc.png">

### 2. Lifetime
#### 2.1 A most plausible lifetime 
Lifetime of transient phenomenon such as the duration of OH masers associated with evolved stars <br>
can be estimated by consideing the probability 
<img src="https://latex.codecogs.com/gif.latex?P^m_n"/>
of detecting *m* dissapeared maser sources in *n* samples in <br> 
a given amount of time; *dT* as demonstrated by [Engels and Jimenes-Esteban (2007)](https://arxiv.org/pdf/0710.1697.pdf). <br>
<img src="https://latex.codecogs.com/gif.latex?P_m^n&space;=&space;\frac{n!}{m!(n-m)!}\big(\frac{\delta&space;T}{T}\big)^m&space;\big(1&space;-&space;\frac{\delta&space;T}{T}\big)^{n-m}"/>
For example if you have *m*=3 maser sources vanished in *n*=100 samples after *dT*=10 years, you will get the most prausible lifetime 333 years.
```
import numpy as np
from scipy.special import comb
import lifetime_scaleheight.lifetime_scaleheight as ls

M    = 30
N    = 445
DT   = 20
SPAN = 1000
lt   = sl.LifeTime(M,N,DT,SPAN)
lifetime = lt.lifetime()
print(format(lifetime,'.1f'))
>>> '296.7'
```
#### 2.2 Upper limits of a lifetime
Meanwhile, the upper limit of OH maser lifetime 
<img src="https://latex.codecogs.com/gif.latex?Q^m_n"/>
with different significance levels can be estimated as follows

<img src="https://latex.codecogs.com/gif.latex?Q_m^n&space;=&space;\sum_{i=m}^n&space;\frac{n!}{m!(n-m)!}\big(\frac{\delta&space;T}{T}\big)^m&space;\big(1&space;-&space;\frac{\delta&space;T}{T}\big)^{n-m}"/>.

The upper limits for lifeitme of OH masers assuming at least *m*=3 maser sources vanish in <br>
*n*=100 samples in *dT*=10 years will be 505, 1258, 2000 years depending on significance levels (1, 2, 3σ).

```
lifetime_upper_limit = lt.upper_limit()
print(lifetime_upper_limit)
>>> [326.90000000000435, 411.90000000000555, 512.900000000007] # 1 sigma, 2 sigma, 3 sigma
```
#### 2.3 Quick visualization of the probability distribution 
```
#  Pnm
xscale = 'linear'
type   = 'lifetime'
lt.visualize(xscale,type)
```
<img width="300" alt="lifetime" src="https://user-images.githubusercontent.com/49733387/101880602-89733100-3bd6-11eb-8a1e-4498e7994929.png">

```
# Qnm
xscale = 'linear'
type   = 'upper_limit'
lt.visualize(xscale,type)
```
<img width="300" alt="upperlimit" src="https://user-images.githubusercontent.com/49733387/101880623-909a3f00-3bd6-11eb-8df5-a1b2a5023c2e.png">
