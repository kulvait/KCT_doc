<!--
.. title: Working with KCT CBCT 5 Parallel beam geometry
.. slug: working-with-kct-cbct-5-parallel-beam-geometry
.. date: 2022-03-11 11:09:24 UTC+01:00
.. tags: using_kct_blog
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

In the [previous chapter](link://slug/working-with-kct-cbct-2-projective-geometry-and-camera-matrices-to-describe-ct-geometry) we learned about cone beam geometry and projective matrices. For synchrotron applications might be convenient to work also with parallel ray geometry. Here we review the concept of the projection matrices to define this type of geometry.

# Defining parallel rays geometry

In [ASTRA toolbox](https://www.astra-toolbox.com/docs/geom3d.html#projection-geometries) parallel ray geometry in 3D is described by 12 numbers representing four 3D vectors.

Vector    | Description
----------|----------------------------------------------
ray       | ray direction
d         | the center of the detector
u         | the vector from detector pixel (0,0) to (0,1)
v         | the vector from detector pixel (0,0) to (1,0)

Additional two integers need to be supplied representing detector size

Parameter                   | Description
----------------------------|----------------------------------------------
projection-sizex            |  X dimension of detector in pixel count.
projection-sizey            |  Y dimension of detector in pixel count.

It is straightforward to see that such description is sufficient to fully describe the setup but it might not be as straightforward what is the simplest way how to project point onto the detector. Because for given point $x$ all the $x + t ray$ points shall be projected to the same point, we have to orthogonalize $u$ and $v$ with respect to $ray$.

$uo = u - (u, ray)/(ray,ray) ray$ 
$vo = u - (u, ray)/(ray,ray) ray$ 

Now we have to project center of the detector onto the projector

$PXd0 = (d, u0)$
$PYd0 = (d, v0)$

We know that $PXd0$ shall correspond to the $PX=0.5projection-sizex$ and $PYd0$ shall correspond to the $PY=0.5projection-sizey$. Therefore the formula to get PX and PY for arbitrary point x will read

$PX = (x, u0)-PXd0+0.5projection-sizex$
$PY = (x, v0)-PYd0+0.5projection-sizey$.

You can imediatelly see that we have affine transformation and that we can create $4 \times 2$ projection matrix, which for given x project it to the position on the detector. Moreover when instead of center of the detector we use zero of the detector, the projection matrix will be independent of projector dimensions.

Another observation is that we are not able to recover vectors $u$ and $v$ from the projection matrix. The reason is that the matrix is based on vectors $u0$ and $v0$, which are orthogonal to the incomming rays. Therefore if we need the information about the tilt of the detector, we need to provide it separatelly. 


# Projection matrices

It is tempting to use projection matrices as they provide full correspondence between 3D and 2D objects. We use just afine projection of 3D space into 2D. On the other hand in this formalism is not possible to encode tilt of the detector and it is therefore needed to expect either that there is no tilt or to provide this information by different means.
