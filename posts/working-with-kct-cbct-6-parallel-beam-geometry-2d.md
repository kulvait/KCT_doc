<!--
.. title: Working with KCT CBCT 6 Parallel beam geometry 2D
.. slug: working-with-kct-cbct-6-parallel-beam-geometry-2d
.. date: 2023-03-02 11:09:24 UTC+01:00
.. tags: using_kct_blog
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
.. status: private
-->


In the [last chapter](link://slug/working-with-kct-cbct-5-parallel-beam-geometry) I have introduced concept of afine transform represented by projection matrix for parallel beam geometry for 3D -> 2D transform. Here I will deal with specific case of 2D -> 1D transform. It is just simplification of previous chapter, but because of its importance it is good to have it here.

# When to use PBCT2D geometry

In many use cases, we have very aligned volume to the projector grid. So that the PX lines on the detector and VX, VY planes of the volume are both parallel to the incomming rays. Moreover, the detector pixel width is the same as the width of the volume slice and there is 1:1 correspondence between them. PBCT projector is not able to take speed advantage of simplified geometry setup. This is the case, where PBCT2D projector comes into the play. Projector is implemented in `opencl/pbct2d_cvp.cl` as an exact cutting voxel projector.


# Defining parallel rays geometry

The affine transform, which takes 2D vector and project it into 1D line can be simply defined as
$$P(x) = a_1 x_1 + a_2 x_2 + P0,$$
where $P(0) = P0$. So in this case it is sufficient to describe geometry by three numbers. 

Let's take 


Parallel rays geometry is simply projecting 3D points onto 2D plane. Here we review how the parallel rays geometry is encoded in other tools and if we can use the idea of projection matrices to describe it.


In [ASTRA toolbox](https://www.astra-toolbox.com/docs/geom3d.html#projection-geometries) parallel ray geometry in 3D is described by 12 numbers representing four 3D vectors.

|Vector    | Description                                  |
|----------|----------------------------------------------|
|r         | ray direction                                |
|d         | the center of the detector                   |
|$v_x$     | the vector from detector pixel (0,0) to (0,1)|
|$v_y$     | the vector from detector pixel (0,0) to (1,0)|

Additional two integers need to be supplied representing number of pixels in each detector direction.

|Parameter                   | Description                                  |
|----------------------------|----------------------------------------------|
|NX                          |  X dimension of detector in pixel count.     |
|NY                          |  Y dimension of detector in pixel count.     |


It is straightforward to see that such description is sufficient to fully describe the setup. Now we need to use this description to project point $x = (x_1, x_2, x_3)$ onto the detector by means of transform $P(x) = (PX, PY)$. For a parameter $t$ we know that the projection of all points $x + t \cdot r$ shall be the same for all $t \in \mathbb{R}$. Let's try to model parallel beam projection in 3D to the 2D detector by means of afine transform so that the coordinates on the detector $PX(x)$ and $PY(x)$ will be

$$PX(x) = PX0 + a_1 x_1 + a_2 x_2 + a_3 x_3,$$
$$PY(x) = PY0 + b_1 x_1 + b_2 x_2 + b_3 x_3,$$

where $PX(0) = PX0$ and $PY(0) = PY0$. This description requires to know 8 numbers represented by means of $4 \times 2$ projection matrix, which for given $x$ projects it to the position on the detector position $P(x) = (PX(x), PY(x))$.

Now having Astra representation of the geometry, what the values defining afine transform shall be? Since $P(x + t \cdot r) = P (x)$ the vectors $a$ and $b$ must be orthogonal to $r$. We also know that $PX(v_x) - PX(0) = 1$ and $PY(v_y) - PY(0) = 1$ so from other algebraic consideration we conclude that $a$ and $b$ will be multiples of orthogonalized vectors

$$v_x^0 = v_x - (v_x, r)/(r,r) r,$$ 
$$v_y^0 = v_y - (v_y, r)/(r,r) r.$$ 

In particular

$$a = v_x^0/(v_x^0,v_x^0),$$
$$b = v_y^0/(v_y^0,v_y^0).$$

When we project center of the detector $d$ onto the detector obviously we shall obtain center of it

$$PX(d) = PX0 + (d, a) = 0.5 NX,$$
$$PY(d) = PY0 + (d, b) = 0.5 NY$$

therefore 

$$PX0 = 0.5 NX - (d,a),$$
$$PY0 = 0.5 NY - (d,b).$$

Therefore the formula to get PX and PY for arbitrary point x using Astra geometry will be

$$PX(x) = (x, a)-(d, a)+0.5 NX,$$
$$PY(x) = (x, b)-(d, b)+0.5 NY.$$

When instead of center of the detector we use the origin of the detector, the projection matrix will be independent of projector dimensions, therefore the classes `Geometry3DParallel` and `Geometry3DParallelCameraMatrix` have the constructor analogous to ASTRA, where instead of the center of the detector there is detector origin. Detector origin is a point on the detector with coordinates PX=PY=0 and by convence it is in the center of corner pixel. We use the following initialization.

|Vector    | Description                                                                          |
|----------|--------------------------------------------------------------------------------------|
|r         | ray direction                                                                        |
|o         | the origin of the detector, point (0,0) by convention at the center of (0,0) pixel   |
|u         | the vector from detector pixel (0,0) to (0,1)                                        |
|v         | the vector from detector pixel (0,0) to (1,0)                                        |

It holds that

$$PX(o) = PX0 + (o, a) = 0,$$
$$PY(o) = PY0 + (o, b) = 0.$$

Therefore we have $PX0 = -(o,a)$, $PY0 = -(o,b)$ and
$$PX(x) = (x, a)-(o, a),$$
$$PY(x) = (x, b)-(o, b).$$

Another observation is that we are not able to recover vectors $v_x$ and $v_y$ from the projection matrix or the transformation $P$. The reason is that the matrix is based on vectors $v_x^0$ and $v_y^0$, which are orthogonal to the incomming rays. Therefore if we need the information about the tilt of the detector, we need to provide it separatelly. For all applications shall be sufficent to know the cosine of the angle between the detector and surface orthogonal to incoming rays, which is usually 1.

# Projection matrices

For projection of the point x we use homogeneous coordinates, where we represent $x = (x_1, x_2, x_3, 1)$ and the 3D parallel projection matrix
$$
\mathbf{P} = \begin{pmatrix}
a_1&a_2&a_3&px_0 \\\
b_1&b_2&b_3&py_0
\end{pmatrix},
$$
which represents the projection onto the detector orthogonal to the incomming rays with particular origin. It is obvious that for each tilted detector there must be one virtual orthogonal detector to the incomming rays. Let's recover what are the vectors $v_x=v_x^0$, $v_y=v_y^0$, the ray direction $r$ and origin $o$ of the virtual detector corresponding to the matrix $\mathbf{P}$.

Vector $r$ or homogeneous $(r,0)$ needs to be orthogonal to the rows of matrix $\mathbf{P}$ since $\mathbf{P} (x,1)^\top$ = $\mathbf{P} (x,1)^\top + t (r,0)^\top$ for all $t \in \mathbb{R}$. Such vector is
$$r = \frac{a \times b}{\|a \times b\|}.$$ 

The vectors $v_x$ and $v_y$ have the property $(v_x, a) = 1$, $(v_x,b) = 0$, $(v_x,r)=0$, $(v_y,b) = 1$, $(v_y, a) = 0$, $(v_y, r)=0$. Therefore $v_x$ and $v_y$ are scaled vectors $a$ and $b$, namely
$$v_x = \frac{a}{(a,a)},$$  
$$v_y = \frac{b}{(b,b)}.$$  

Now remains to identify vector $o$ corresponding to the origin. In fact there is a line, which projects to (0,0) so that best will be to obtain minimum norm solution, which will be a linear combination of vectors a and b, whose linear span creates a subspace of dimension two, that in turn means that there exist unique decomposition of $o = \alpha a + \beta b$ for which it holds that
$\mathbf{P} (o,1)^\top = (0,0)^\top$. For orthogonal $a$ and $b$ is the situation simplest but let's assume these vectors are not orthogonal we end up with the system
$$ \begin{pmatrix}
(a,a) & (a,b) \\\
(b,a) & (b,b)
\end{pmatrix}
\begin{pmatrix}
\alpha \\\
\beta
\end{pmatrix}
=
-\begin{pmatrix}
px_0 \\\
py_0
\end{pmatrix}
$$
and the solution
$$ 
\begin{pmatrix}
\alpha \\\
\beta
\end{pmatrix}
=
-\begin{pmatrix}
(a,a) & (a,b) \\\
(b,a) & (b,b)
\end{pmatrix}^{-1}
\begin{pmatrix}
px_0 \\\
py_0
\end{pmatrix}
.$$

Finally we conclude $o = \alpha a + \beta b$, which completes the correspondence between projective operator given as matrix and explicit declaration of the geometry in ASTRA style. Note that previous equation can be also used for establishing inverse projective operator and its minimum norm solution. Instead of (0, 0) we can use any other point on the projection plane.
