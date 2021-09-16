<!--
.. title: Working with KCT CBCT 3 Python implementation of circular CT trajectory
.. slug: working-with-kct-cbct-3-python-implementation-of-circular-ct-trajectory
.. date: 2021-09-15 12:05:50 UTC+02:00
.. tags: using_kct_blog
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

In the last post we build the mathematical foundation how to construct camera matrices for FDCT setup. Now we use this knowledge to create DEN file with a circular geometry and show how to work with it in the KCT package.

## System setup
We have yet mastered the theory so let's produce the set of camera matrices for given trajectory.
I have put a script that implements what has been said [to github](https://github.com/kulvait/KCT_scripts/blob/master/createCameraMatricesForCircularScanTrajectory.py).
Version of [denpy](https://github.com/kulvait/KCT_denpy) package must be at least 1.1.2 to have function for storing double DEN `storeNdarrayAsDoubleDEN`.

For this demonstration to run on your computer you first need to install `denpy` package and clone [DEN scripts](https://github.com/kulvait/KCT_scripts) package. Let's suppose you are running Debian 11. 

You can create a folder `KCT` and `KCT_bin` in your home directory. Go into this folder and clone KCT scripts into the folder scripts.
```
git clone https://github.com/kulvait/KCT_scripts.git scripts
```
Then into your .bashrc you can add the following
```
export PATH=$PATH:~/KCT_bin::~/KCT_scripts
```
Then you shall also install [denpy](https://github.com/kulvait/KCT_denpy)

```
pip3 install --user git+https://github.com/kulvait/KCT_denpy.git
```

if you need to upgrade run

```
pip3 install --user --upgrade git+https://github.com/kulvait/KCT_denpy.git
```

Now you have installed everything to produce the camera matrices. Optionally you might need to install KCT dentk package, that will be very useful for working with the files in [DEN format](link://slug/den-format).

Go to the KCT folder and clone it

```
git clone https://github.com/kulvait/KCT_dentk dentk
```
Then follow the instructions in README file at https://github.com/kulvait/KCT_dentk to build the package.

## Python implementation

When the system is set up, we can just run
```
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file CM.den
```
as I set all the default parameters of this script according to [last post](link://slug//working-with-kct-cbct-2-projective-geometry-and-camera-matrices-to-describe-ct-geometry). You can see that the den file `CM.den` and the text file `CM.den.params` were produced. Params file is not apriori needed for working with KCT but it is a good practice to know how your files were created.

Let's have a look to the main part of the script
```python
I=ARG.source_to_isocenter
A=ARG.source_to_detector
M=float(ARG.projection_sizey)
N=float(ARG.projection_sizex)
PX=ARG.pixel_sizex
PY=ARG.pixel_sizey
VIEWCOUNT = ARG.number_of_angles
OMEGA = ARG.omega_zero
OMEGAINCREMENT = 2*np.pi/VIEWCOUNT

#Let's create specified set of projection matrices as np.array
CameraMatrices = np.zeros((3,4,0), dtype=np.float64)

for viewIndex in range(VIEWCOUNT):
	s = sourcePosition(OMEGA, I) 
	_A3=A3(ARG.pixel_offsetx, ARG.pixel_offsety)
	_A2=A2(M, N)
	_A1 = A1(PX, PY)
	_E = E(A)
	_X2 = X2(s) 
	_X1 = X1(OMEGA)
	CM = _A3.dot(_A2).dot(_A1).dot(_E).dot(_X2).dot(_X1)
	CameraMatrices = np.dstack((CameraMatrices, CM))
	OMEGA += OMEGAINCREMENT

DEN.storeNdarrayAsDoubleDEN(ARG.outputMatrixFile, CameraMatrices, ARG.force)
```

First we initialize the parameters.

Then we prepare the numpy matrices according to [the last post](link://slug//working-with-kct-cbct-2-projective-geometry-and-camera-matrices-to-describe-ct-geometry) and finally we do the multiplication and concatenate the matrices.

On the last line you can see how easy is to convert `numpy.ndarray` into the DEN by one call. 

Using the script, we produce the camera matrices we need for given trajectory setup.
To remind you the parameters let's have a look to the `CM.den.params` file

```json
{
  "_json_message": "Created using KCT script createCameraMatricesForCircularScanTrajectory.py",
  "force": true,
  "number_of_angles": 360,
  "omega_zero": 0,
  "outputMatrixFile": "CM.den",
  "pixel_offsetx": 0.0,
  "pixel_offsety": 0.0,
  "pixel_sizex": 0.616,
  "pixel_sizey": 0.616,
  "projection_sizex": 616,
  "projection_sizey": 480,
  "source_to_detector": 1198.0,
  "source_to_isocenter": 749.0,
  "write_params_file": true
}
```
So that we can check which particular parameters were used to create our trajectory. To dig deeper, I have written `dentk-mathinfo` as part of the dentk package. We can check decompose our matrices and check properties such as source position of first few frames by running 

```bash 
dentk-matinfo CM.den -f 0-5
```

with the following output
```
Camera matrix from 0-th frame:
    |   -0.200     1.623     0.000   149.737|                     | 1944.805    -0.000   239.500| |    0.000     1.000     0.000|    0.000|
P = |   -0.257     0.000    -1.623   192.252| = C[Q|u] =    1198.0|    0.000  1944.805   307.500|.|   -0.000    -0.000    -1.000|    0.000|
    |   -0.001     0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -1.000     0.000    -0.000|  749.000|
S = [749.00, -0.00, -0.00], -Q^T u = [749.00,  0.00,  0.00].
Camera matrix from 1-th frame:
    |   -0.228     1.620     0.000   149.737|                     | 1944.805     0.000   239.500| |   -0.017     1.000    -0.000|    0.000|
P = |   -0.257    -0.004    -1.623   192.252| = C[Q|u] =    1198.0|    0.000  1944.805   307.500|.|   -0.000    -0.000    -1.000|    0.000|
    |   -0.001    -0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -1.000    -0.017    -0.000|  749.000|
S = [748.89, 13.07, -0.00], -Q^T u = [748.89, 13.07,  0.00].
Camera matrix from 2-th frame:
    |   -0.256     1.615     0.000   149.737|                     | 1944.805     0.000   239.500| |   -0.035     0.999    -0.000|    0.000|
P = |   -0.257    -0.009    -1.623   192.252| = C[Q|u] =    1198.0|    0.000  1944.805   307.500|.|   -0.000     0.000    -1.000|    0.000|
    |   -0.001    -0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -0.999    -0.035    -0.000|  749.000|
S = [748.54, 26.14, -0.00], -Q^T u = [748.54, 26.14,  0.00].
Camera matrix from 3-th frame:
    |   -0.285     1.611     0.000   149.737|                     | 1944.805     0.000   239.500| |   -0.052     0.999    -0.000|    0.000|
P = |   -0.256    -0.013    -1.623   192.252| = C[Q|u] =    1198.0|    0.000  1944.805   307.500|.|   -0.000     0.000    -1.000|    0.000|
    |   -0.001    -0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -0.999    -0.052    -0.000|  749.000|
S = [747.97, 39.20, -0.00], -Q^T u = [747.97, 39.20,  0.00].
Camera matrix from 4-th frame:
    |   -0.313     1.605     0.000   149.737|                     | 1944.805     0.000   239.500| |   -0.070     0.998    -0.000|   -0.000|
P = |   -0.256    -0.018    -1.623   192.252| = C[Q|u] =    1198.0|    0.000  1944.805   307.500|.|    0.000     0.000    -1.000|    0.000|
    |   -0.001    -0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -0.998    -0.070    -0.000|  749.000|
S = [747.18, 52.25, -0.00], -Q^T u = [747.18, 52.25,  0.00].
Camera matrix from 5-th frame:
    |   -0.341     1.600     0.000   149.737|                     | 1944.805     0.000   239.500| |   -0.087     0.996     0.000|   -0.000|
P = |   -0.256    -0.022    -1.623   192.252| = C[Q|u] =    1198.0|    0.000  1944.805   307.500|.|    0.000     0.000    -1.000|    0.000|
    |   -0.001    -0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -0.996    -0.087    -0.000|  749.000|
S = [746.15, 65.28, -0.00], -Q^T u = [746.15, 65.28,  0.00].
```

Here we can see that it was among other things able to show us source positions $S$ for the first views. This seems reasonable when looking to the image of the geometry and how we describe the trajectory. First the source is aligned with $x_1$ axis and it rotates towards $x_2$ axis.

## Next post
In the next post we use created camera matrices, KCT cbct package and downloaded CT volume from public repository and we will show how to project the CT volumes using the FDCT trajectory, which we just created.
