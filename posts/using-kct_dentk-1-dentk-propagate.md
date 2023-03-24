<!--
.. title: Using KCT_dentk 1: dentk-propagate
.. slug: using-kct_dentk-1-dentk-propagate
.. date: 2023-03-23 14:56:39 UTC+01:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

This post will be about `dentk-propagate` a new part of dentk toolkit, I use it as a script when developing the tool to have some important notes here.

It is implementation of the Fresnel and Rayleigh-Sommerfeld propagators intended to be used in scope of the algorithms such as [Gerchberg-Saxton](http://www.u.arizona.edu/~ppoon/GerchbergandSaxton1972.pdf).  

## Theory

We assume there is an initial plane wave given by the complex function $U(x,y,z)$, which is called phasor in [Introduction to Fourier Optics](https://openlibrary.org/books/OL22748959M/Introduction_to_Fourier_optics) and we know the function $U(x,y,0)$. We ca then obtain 
$$U(x,y,z) = U(x,y,0) * p^z(x,y)$$
values of $U$ at particular $z$.

We assume that we deal with the plane waves of the form
$$U = E e^{ikz},$$
where $E$ is complex envelope of the wave.
 
### Fresnel propagator
So now we are in the $\mathbb{R}^3$ space. Fresnel propagator takes the form
$$p_F(x,y) = \frac{e^{ikz}}{i \lambda z} e^{i k \frac{x^2+y^2}{2z}}$$
it can be shown that after Fourier transformation $P_F(\xi_x,\xi_y = \mathcal{F}(p_F))$ we get
$$P_{F}(\xi_x, \xi_y) = \frac{e^{ikz}}{i \lambda z}  \frac{i 2 z\pi}{ k} e^{-i\frac{2 z\pi^2 (\xi_x^2 + \xi_y^2)}{k}} =  e^{ikz}  e^{-i \lambda z \pi (\xi_x^2 + \xi_y^2)}.$$

This is very important formula as we can also use it to transform envelope as $e^{ikz}$ is a prefactor.

### Rayleigh-Sommerfeld propagator
In case or this propagator, which is constructed using less assumptions than Fresnel we have that,
$$p_H(x,y) = \frac{1}{i \lambda} \frac{z e^{i k \sqrt{x^2+y^2+z^2},}}{x^2+y^2+z^2}.$$

Interestingly, when neglecting evanescent waves, we obtain also the formula for Fourier transformed function
$$P_H(\xi_x, \xi_y) =
\begin{cases} e^{i k z  \sqrt{1 - (\lambda \xi_x)^2 - (\lambda \xi_y)^2}}, & (\lambda \xi_x)^2 + (\lambda \xi_y)^2 < 1  \\
0,&  (\lambda \xi_x)^2 + (\lambda \xi_y)^2 \geq 1.\end{cases}$$

Now we see that it is not that easy to separate prefactor in $U$ expression and thus some computations might be tricky. You can also see, neglecting second case, under first order Taylor expansion
$\sqrt{1-x} = 1 - x/2$, the Rayleigh-Sommerfeld propagator becomes Fresnel propagator in Fourier space.

## Discretization

Now the problem is that we usually measure intensity $I$ or the extinction $EXT$ but not $U(x,y,0)$. It is convenient to write for plane wave
$$U = E e^{ikz}$$ 
and further decompose complex envelope
$E = A e^{i\Phi} = \sqrt{I} e^{i\Phi},$
where $I$ is positive real intensity, $\Phi$ is real phase shift and $A = \sqrt{I}$ is positive real amplitude.

### Inputs
The inputs of `dentk-propagate` are functions $I$ and $\Phi$ defining complex envelope $E$

`input_intensity`

`input_phase`

`output_intensity`

`output_phase`

If you were interested, I use euler formula to construct complex envelope by the following code
```
        float intensity, phase, amplitude;
        IDX = dimx * PY + PX;
        IDX_padded = dimx_padded * PY + PX;
        intensity = GPU_intensity[IDX]; 
        phase = GPU_phase[IDX];
        amplitude = sqrt(intensity);    
        v.x = amplitude * cosf(phase);  
        v.y = amplitude * sinf(phase);  
        GPU_envelope[IDX_padded] = v;   

```
But let's leave implementation for now and talk a bit about discretization.

### Convolution theorem

It is of course tempting to use Fourier convolution theorem in order to compute the outlined convolution
$$U(x,y,z) = U(x,y,0) * p^z(x,y)$$
or rather
$$E(x,y,z) = E(x,y,0) * p^z(x,y) e^{-ikz} $$
by just
$$\mathcal{F} E(\xi_x,\xi_y,z) = \mathcal{F} E(x,y,0) . P^z(\xi_x, \xi_y) e^{-ikz} $$

Do you see how nicely we can use the fact $e^{ikz} e^{-ikz} = 0$ to get rid of the space vibrations across $z$ in the wave $U$ by working with $E$. Especially for Fourier, it comes handy and it is one of the main reason why Fresnel propagator is implemented more often than Rayleigh-Sommerfeld.

Now however, we have a serious issue with the discretization. In fact, we don't have continuous function but discretized values of $E$ typically on some kind of detector. Again it is tempting to use discrete Fourier transform to do convolution by multiplication but we have to make it right. And we can see in a few moments there are shades of rightness in this involved.

I would like to mention three papers, which deal with this kind of discretization/sampling problems. The paper [Katkovnik2008](https://doi.org/10.1364/AO.47.003481) introduces so called discrete diffraction transform assuming that the underlying field is stepwise constant on the area of the pixels and the papers [Onural2007](https://doi.org/10.1364/JOSAA.24.000359) and [Onural2004](https://doi.org/10.1117/1.1802232) deal with the question what if the values on the pixels are just samples of underlying function. 

At the beggining, I start implementing problem more according the Katkovnik2008 since it is simpler. In some sense authors do something simmilar, what footprint projectors or my cutting voxel projector do in scope of CT projection. Do not consider just the value of the kernel at the center of the pixel but integrate it over the whole pixel area. This is additional work but at least for Fresnel diffraction, it shall be doable to implement such approach and to be honest, I don't know yet if it will bring any improvement. What is important however, that even when we approximate the integral as a central value times the area of the pixel, we have nice theory how discretization shall work and a nice foundation how to transfer continuous to the discrete case. So this result will perform here as a check, that my implementation is reasonably correct as in the first case, I would like to use precomputed Fourier kernels of propagators.

### Sampled frequency space
It is natural for me to understand how we can go from continuous to discrete space with the values of envelope. We just know the centers of the pixels are `--pixel-sizex` and `--pixel-sizey` apart. But what I am struggling with is a sampling in frequency space since $M \times N$ values will be transferred to $M \times N$ frequencies. But I have already precomputed continuous kernels $P_F$ and $P_H$, which I would like to use. Katkovnik2008 does the integration in a pixel space and not frequency space now the tricky part is the correspondence between continuous frequencies and discretized frequencies and possible scaling of the kernels. 

For a moment, forget about the sampling errors and just relate definitions of the continuous and discrete Fourier transforms and their inversions and just assume we have function with the support on $[0,\delta N)$ that is constant on $[n\delta, (n+1)\delta)$ and zero elsewhere. Fourier transform of such function will look as follows 
$$F(\xi) = \int_{-\inf}^{\inf} f(x) e^{-i 2 \pi x \xi} \mathrm{d}x \approx \delta \sum_{n=0}^{N-1} f(n \delta) e^{-i2 \pi n \delta \xi}.$$

This is in a nutshell what we do when we approximate continuous Fourier transform by the discrete, note that the $\approx$ is in fact equality for pixelwise constant function, but we don't believe underlying function is really pixelwise constant. But wait, this formula looks simmilar as a formula for discrete Fourier transform

$$F_k = \sum_{n=0}^{N-1} f_n e^{-i2 \pi n k/N},$$

when we set $f_n = f(\delta n)$ we also have $F_k = \frac{1}{\delta} F(\frac{k}{n \delta})$.

Wow! This shall be recepy how to manipulate with kernel for the convolution! Now if you have a look to the actual implementation e.g. in CUDA kernel `cuda/diffractionPhysics.cu` [spectralMultiplicationFresnel](https://github.com/kulvait/KCT_dentk/blob/48199e8a8a65d534417f585c4351475055399f23/cuda/diffractionPhysics.cu#L143) you can see that there is no division by pixel area. Why? It is because of the convolution. If we do that integral, we have to multiply by the pixel area each discretized cell to obtain correct result. This is the reason why in eq (14) in Katkovnik2008 there is multiplication by pixel area. So instead of dividing and then multiplying, we just omit the operation.

Now this is a standard implementation of the discretized diffraction. On my to do list is first to implement computation of the integral (24) in Katkovnik2008 and implement more precise Fresnel propagator. This computation is relatively tricky as it involves evaluating complex error function. There is `C++` implementation of [Faddeeva package](http://ab-initio.mit.edu/wiki/index.php/Faddeeva_Package) which is to be considered for this task. For now it seems however, that using analytically precomputed kernels is better than simply computing kernel in real space and converting it to Fourier space using FFT. The implementation in Katkovnik2008 require however doing this, because averaging is not in frequency space. Another point on my todolist is to consider sampling effects and possible corrections in scope of works Onural2007 and Onural2004.

## Implementation

We assume a plane wave that is known at the distance $z=0$ and we would like it to propagate it to the distance $z>0$ or $z<0$, to do so it is neccessary to have the following parameters 

### wave-energy
This parameter is in keV. Although the formulas needs usually just the wavelength $\lambda$, it is better to use some natural unit of the typical wave energy. Visible light is something as $0.002$ keV. Typical higher energy wave can have $20$ keV, so we set it as default. 

### Existing outputs

Many programs in `dentk` behaves differently when outputs exist and force is specified and frames is specified as well not coverring whole volume from the case output does not exist. 

If output exists and they are of the same dimensions and element types as inputs, when there is frames argument specified, output z indices specified by that argument will be overvritten.

If output does not exist and frames vector is specified, output dimz is equal to the size of the frames vector and outputs are written in order in which they are specified.

### Zero padding

I would like to perform convolution using convolution theorem. Now the problem is I need to pad it not to have cyclic convolution but linear convolution. Especially for the object with the nonzero phases on the boundary it might create phantom diffraction patterns, so symmetric padding is on the todo list.

### Code efficiency

There can be bottleneck that each thread allocates and frees cuda memory. It would be probably more efficient if these block were preallocated and each thread uses its own block. There is also a question of the multiGPU implementation.
