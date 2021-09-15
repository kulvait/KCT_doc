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

Before we define particular geometry corresponding to the flat panel detector CT trajectory, we need to know some theory about projective geometry and camera matrices. This will be the content of this post.

## Python implementation
We have yet mastered the theory so let's produce the set of camera matrices for given trajectory.
I have put a script that implements what has been said [to github](https://github.com/kulvait/KCT_scripts/blob/master/tutorials/createProjectionMatrices.py).
Version of denpy package must be at least 1.1.2 to have function for storing double DEN `storeNdarrayAsDoubleDEN`.

Using the script, we produce the camera matrices we need for given trajectory setup.
We can check their properties such as source position by running `dentk-matinfo` with the following output
```
dentk-matinfo CM.den -f 0-3
Camera matrix from 0-th frame:
    |   -0.257     1.623     0.000   192.252|                     | 1944.805    -0.000   307.500| |    0.000     1.000     0.000|    0.000|
P = |   -0.200     0.000    -1.623   149.737| = C[Q|u] =    1198.0|    0.000  1944.805   239.500|.|   -0.000    -0.000    -1.000|    0.000|
    |   -0.001     0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -1.000     0.000    -0.000|  749.000|
S = [749.00,  0.00, -0.00], -Q^T u = [749.00,  0.00,  0.00].
Camera matrix from 1-th frame:
    |   -0.285     1.619     0.000   192.252|                     | 1944.805     0.000   307.500| |   -0.017     1.000    -0.000|    0.000|
P = |   -0.200    -0.003    -1.623   149.737| = C[Q|u] =    1198.0|    0.000  1944.805   239.500|.|   -0.000    -0.000    -1.000|    0.000|
    |   -0.001    -0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -1.000    -0.017    -0.000|  749.000|
S = [748.89, 13.07, -0.00], -Q^T u = [748.89, 13.07,  0.00].
Camera matrix from 2-th frame:
    |   -0.313     1.613     0.000   192.252|                     | 1944.805     0.000   307.500| |   -0.035     0.999    -0.000|    0.000|
P = |   -0.200    -0.007    -1.623   149.737| = C[Q|u] =    1198.0|    0.000  1944.805   239.500|.|   -0.000     0.000    -1.000|    0.000|
    |   -0.001    -0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -0.999    -0.035    -0.000|  749.000|
S = [748.54, 26.14, -0.00], -Q^T u = [748.54, 26.14,  0.00].
Camera matrix from 3-th frame:
    |   -0.341     1.608     0.000   192.252|                     | 1944.805     0.000   307.500| |   -0.052     0.999    -0.000|    0.000|
P = |   -0.200    -0.010    -1.623   149.737| = C[Q|u] =    1198.0|    0.000  1944.805   239.500|.|   -0.000     0.000    -1.000|    0.000|
    |   -0.001    -0.000     0.000     0.625|                     |    0.000     0.000     1.000| |   -0.999    -0.052    -0.000|  749.000|
S = [747.97, 39.20, -0.00], -Q^T u = [747.97, 39.20,  0.00].
```

Here we can see that it was able to show us source positions $S$ for first three views. This seems reasonable when looking to the image of the geometry and how we describe the trajectory. First the source is aligned with $x_1$ axis and it rotates towards $x_2$ axis.

## Next post
In the next post we use created camera matrices, KCT cbct package and downloaded CT volume from public repository and we will make projections.
