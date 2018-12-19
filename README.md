
![logo](https://github.com/EnvModellingGroup/hdrs/blob/master/docs/logo_small.png)


About hrds
===========
hrds is a python package for obtaining points from a set of rasters at 
different resolutions.
You can request a point and hrds will return a value based on
the highest resolution dataset (as defined by the user) available at that point, blending
datasets in a buffer region to ensure consistency.

[![Build Status](https://travis-ci.org/EnvModellingGroup/hdrs.svg?branch=master)](https://travis-ci.org/EnvModellingGroup/hdrs)

Current release:
[![DOI](https://zenodo.org/badge/155502078.svg)](https://zenodo.org/badge/latestdoi/155502078)

Prerequisites
---------------
* python 2.7 or 3.
* numpy
* scipy
* osgeo.gdal (pygdal) to read and write raster data

These instruction assume a Debian-based Linux. HDRS should work on other systems, but is currently untested.

To install pygdal, install the libgdal-dev packages and binaries, e.g.

```bash
sudo apt-get install libgdal-dev gdal-bin
```

To install pygdal, we neeed to check which version of gdal is installed:
```bash
gdal-config --version
```

Install using pip, using the correct version as gleaned from the command above. Note you may need to 
increase the minor version number, e.g. from 2.1.3 to 2.1.3.3.

```bash
pip install pygdal==2.1.3.3
```
Replace 2.1.3.3 with the output from the ``gdal-config`` command.

To use this in your Firedrake environment, remember to do the last step after
activating the Firedrake environment.

You can install HRDS using the standard:
```bash
python setup.py install
```

Functionality
---------------
* Create buffer zones as a preprocessing step if needed
* Obtain value at a point based on user-defined priority of rasters

Example of use via [thetis](http://thetisproject.org/):
```python
from firedrake import *
from thetis import *
from firedrake import Expression
import sys
sys.path.insert(0,"../../")
from hrds import HRDS

mesh2d = Mesh('test_mesh.msh') # mesh file

P1_2d = FunctionSpace(mesh2d, 'CG', 1)
bathymetry2d = Function(P1_2d, name="bathymetry")
bvector = bathymetry2d.dat.data
bathy = HRDS("gebco_uk.tif", 
             rasters=("emod_utm.tif", 
                      "inspire_data.tif"), 
             distances=(700, 200))
bathy.set_bands()
for i, (xy) in enumerate(mesh2d.coordinates.dat.data):
    bvector[i] = bathy.get_val(xy)
File('bathy.pvd').write(bathymetry2d)
```

This example loads in an XYZ file and obtains data at each point, 
replacing the Z value with that from HRDS.

```python
import sys
sys.path.insert(0,"../../")

from hrds import HRDS

points = []
with open("test_mesh.csv",'r') as f:
    for line in f:
        row = line.split(",")
        # grab X and Y
        points.append([float(row[0]), float(row[1])])

bathy = HRDS("gebco_uk.tif", 
             rasters=("emod_utm.tif", 
                      "inspire_data.tif"), 
             distances=(700, 200))
bathy.set_bands()

print len(points)

with open("output.xyz","w") as f:
    for p in points:
        f.write(str(p[0])+"\t"+str(p[1])+"\t"+str(bathy.get_val(p))+"\n")
```

This will turn this:
```bash
$ head test_mesh.csv 
805390.592314,5864132.9269,0
805658.162910036,5862180.30440542,0
805925.733505999,5860227.68191137,0
806193.304101986,5858275.05941714,0
806460.874698054,5856322.43692232,0
806728.445294035,5854369.81442814,0
806996.015889997,5852417.19193409,0
807263.586486046,5850464.56943942,0
807531.157082069,5848511.94694493,0
807798.727678031,5846559.32445088,0
```

into this:

```bash
$ head output.xyz 
805390.592314	5864132.9269	-10.821567728305235
805658.16291	5862180.30441	2.721575532084955
805925.733506	5860227.68191	2.528217188012767
806193.304102	5858275.05942	3.1063558741547865
806460.874698	5856322.43692	5.470234157891056
806728.445294	5854369.81443	1.382685066254607
806996.01589	5852417.19193	1.8997482922322515
807263.586486	5850464.56944	4.0836843606647335
807531.157082	5848511.94694	-2.39508079759155
807798.727678	5846559.32445	-2.401006071401176
```

These images show the original data in QGIS in the top right, with each data set using a different colour scheme (GEBCO - green-blue; EMOD - grey; UK Gov - plasma - highlighted by the black rectangle).The red line is the boundary of the mesh used (see figure below). Both the EMOD and UK Gov data has NODATA areas, which are shown as transparent here, hence the curved left edge of the EMOD data.  The figure also shows the buffer regions created around the two higher resolution datasets (top left), with black showing that data isn't used to white where it is 100% used. The effect of NODATA is clear here. The bottom panel shows a close-up of the UK Gov data with the buffer overlayed as a transparancy from white (not used) to black (100% UK Gov). The coloured polygon is the area of the high resolution mesh (see below).

![Input data](https://github.com/EnvModellingGroup/hdrs/blob/master/docs/raster_data_sml.png)

After running the code above, we produce this blended dataset. Note the coarse mesh used here - it's not realistic for a model simulation!

![Blended bathymetry data on the multiscale mesh](https://github.com/EnvModellingGroup/hdrs/blob/master/docs/mesh_bathy_all.png)

If we then zoom-in to the high resolution area we can see the high resolution UK Giv data being used and with no obvious lines between datasets.

![Blended bathymetry data on the multiscale mesh](https://github.com/EnvModellingGroup/hdrs/blob/master/docs/mesh_bathy.png)

Community
-----------

We welcome suggestions for future improvements, bug reports and other issues via the issue tracker. Anyone wishing to contribute code should contact Jon Hill (jon.hill@york.ac.uk) to discuss.

