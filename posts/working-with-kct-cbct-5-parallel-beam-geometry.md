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

Parallel rays geometry is simply projecting 3D points onto 2D plane. Here we review how the parallel rays geometry is encoded in other tools and if we can use the idea of projection matrices to describe it.


In [ASTRA toolbox](https://www.astra-toolbox.com/docs/geom3d.html#projection-geometries) parallel ray geometry in 3D is described by 12 numbers representing four 3D vectors.

|Vector    | Description|
|----------|----------------------------------------------|
|ray       | ray direction|
|d         | the center of the detector|
|u         | the vector from detector pixel (0,0) to (0,1)|
|v         | the vector from detector pixel (0,0) to (1,0)|

Additional two integers need to be supplied representing detector size

|Parameter                   | Description|
|----------------------------|----------------------------------------------|
|NX            |  X dimension of detector in pixel count.|
|NY            |  Y dimension of detector in pixel count.|


It is straightforward to see that such description is sufficient to fully describe the setup. Now we need to use this description to project point $x = (x_1, x_2, x_3)$ onto the detector by means of transform $P(x) = (PX, PY)$. For a parameter $t$ we know that the projection of all points $x + t \cdot ray$ shall be the same for all $t \in \mathbb{R}$. It is natural to model $PX(x)$ and $PY(x)$ by means of affine transform

$$PX(x) = px_0 + a_1 x_1 + a_2 x_2 + a_3 x_3$$
$$PY(x) = py_0 + b_1 x_1 + b_2 x_2 + b_3 x_3$$

and since $P(x + t \cdot ray) = P (x)$ the vectors $a$ and $b$ must be orthogonal to $ray$. We also know that $PX(u) - PX(0) = 1$ and $PY(v) - PY(0) = 1$ so from other algebraic consideration we conclude that $a$ and $b$ will be multiples of orthogonalized vectors

$$u_0 = u - (u, ray)/(ray,ray) ray,$$ 
$$v_0 = v - (v, ray)/(ray,ray) ray.$$ 

In particular

$$a = u_0/(u_0,u_0),$$
$$b = v_0/(v_0,v_0).$$

When we project center of the detector $d$ onto the detector obviously we shall obtain center of it

$$PX(d) = px_0 + (d, a) = 0.5 NX,$$
$$PY(d) = py_0 + (d, b) = 0.5 NY$$

therefore 

$$px_0 = 0.5 NX - (d,a),$$
$$py_0 = 0.5 NY - (d,b).$$

Therefore the formula to get PX and PY for arbitrary point x will read

$$PX(x) = (x, a)-(d, a)+0.5 NX,$$
$$PY(x) = (x, b)-(d, b)+0.5 NY.$$

You can imediatelly see that we have affine transformation and that we can create $4 \times 2$ projection matrix, which for given x project it to the position on the detector. Moreover when instead of center of the detector we use zero of the detector, the projection matrix will be independent of projector dimensions.

Another observation is that we are not able to recover vectors $u$ and $v$ from the projection matrix or the transformation $PX$. The reason is that the matrix is based on vectors $u0$ and $v0$, which are orthogonal to the incomming rays. Therefore if we need the information about the tilt of the detector, we need to provide it separatelly. For all applications shall be sufficent to know the cosinus of the angle between the detector and surface orthogonal to incoming rays, which is usually 1.


# Projection matrices

It is tempting to use projection matrices as they provide full correspondence between 3D and 2D objects. We use just afine projection of 3D space into 2D. On the other hand in this formalism is not possible to encode tilt of the detector and it is therefore needed to expect either that there is no tilt or to provide this information by different means.
