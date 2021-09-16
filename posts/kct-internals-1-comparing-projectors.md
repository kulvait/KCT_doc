<!--
.. title: KCT internals 1 comparing projectors
.. slug: kct-internals-1-comparing-projectors
.. date: 2021-09-20 09:14:20 UTC+02:00
.. tags: kct_internals
.. category: 
.. link: 
.. description: 
.. type: text
.. status: draft
.. has_math: true
-->

In this series of the blog posts I will go into the details of the KCT package that actually do the computations. I would like to convince you that it is also important not to know only interface but how the things work inside as well.

Today I will discuss the issue how to compare different projectors. I have implemented cutting voxel projector (CVP), I think that it is a very nicely structured projector that exploit correspondence between volume integrals of the attenuation and projections. At the end of the day, I have to compare my projector to the established projectors. And how to do it when every projector is original?

I start with the algorithm due to Siddon [Siddon projector](https://doi.org/10.1118/1.595715) from 1984, which is computing exact paths through the voxelized objects. It is interesting that noone was able to came forward with better algorithm then. There are simpler ray casting approaches, but they are more inexact and tend to be slower. This is the only projector in the package that computes attenuation along the rays. Other projectors are trying to somehow estimate what happens, when we cast infinite number of the rays towards each detector pixel.

Let's first priepare camera matrices for the setup analogous to [CBCT geometry post](link:://slug//working-with-kct-cbct-2-projective-geometry-and-camera-matrices-to-describe-ct-geometry) and [Python implementation post](working-with-kct-cbct-3-python-implementation-of-circular-ct-trajectory). 


```bash
#Camera matrices relative to the different binning
PARAMS="--projection-sizex 616 --projection-sizey 480 --pixel-sizex 0.616 --pixel-sizey 0.616"
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS CM4_CIRCULAR.den
```

## Comparing projectors
Now we can compare the different projectors by the way how they project a single voxel on the projector during circular trajectory `CM_CIRCULAR1.den`.

Let's make the volume containing just single voxel with the attenuation value $1$
```bash
dentk-empty 1 1 1 singleUnitVoxel.den --value 1.0
```

Now let's suppose the voxel is centerred at $(0,0,0)$ and its dimensions are $1mm \times 1mm \times 1mm$. Let's do an initial testing projection using Siddon projector
```bash
#Create the directory for projections to keep our working directory cleaner
mkdir -p projections
PARAMS="--voxel-sizex 1.0 --voxel-sizey 1.0 --voxel-sizez 1.0"
OUTPUT="projections/singleUnitVoxel.siddon1"
cllin-projector --force --siddon singleUnitVoxel.den CM4_CIRCULAR.den --projection-sizex 616 --projection-sizey 480 $PARAMS $OUTPUT
```
Now we can see the dot in the center of the projector, which suggest our implementation is correct. To actually compare different projectors actions on the larger grid, we need to increase the size of the voxel when making projections
```bash
PARAMS="--voxel-sizex 100.0 --voxel-sizey 100.0 --voxel-sizez 100.0"
OUTPUT="projections/single100Voxel.siddon1"
cllin-projector --force --siddon singleUnitVoxel.den CM4_CIRCULAR.den --projection-sizex 616 --projection-sizey 480 $PARAMS $OUTPUT
```
Now we see an interesting object, voxel of the cube with $10cm$ edge. Here are now seen even some cone beam effect on the projection data as can be shown in ImageJ.
Let's say now that we shall cast more rays towards each pixel, let's say it will be 100 to have a good approximation 
```bash
PARAMS="--voxel-sizex 100.0 --voxel-sizey 100.0 --voxel-sizez 100.0 --probes-per-edge 100"
OUTPUT="projections/single100Voxel.siddon100"
cllin-projector --force --siddon singleUnitVoxel.den CM4_CIRCULAR.den --projection-sizex 616 --projection-sizey 480 $PARAMS $OUTPUT
```
Now you can see the projection gets slower. Its because towards each detector pixel is now casted $100 \times 100 = 10.000$ rays. And there is $616 \times 480$ pixels and $360$ views. So in total around one trilion (12 zeros, so that 1,000,000,000,000) rays is casted.


I'd like to mention that in `cllin-projector` is the running time of the projections much slower than the running time of the projection in the reconstruction program `cllin-krylov`. The resason is that we typically need this for debugging or one time reprojection and it is written to allow me faster debugging. The place, where the projectors and backprojectors are optimized for the speed is the reconstruction by means of `cllin-krylov`.

Now we have our very fine approximation of the actual shape of the voxel projection `single100Voxel.sidon100`. However how do we compare it to `single100Voxel.sidon1`. How to do it? First make a difference 
```bash
dentk-calc --subtract single100Voxel.siddon1 single100Voxel.siddon100 single100Voxel.siddon1_minus_siddon100
```
Now we have to debug!
```bash
PARAMS="--voxel-sizex 100.0 --voxel-sizey 100.0 --voxel-sizez 100.0 --probes-per-edge 100 --frames 90"
OUTPUT="projections/single100Voxel.siddon100_90"
cllin-projector --force --siddon singleUnitVoxel.den CM4_CIRCULAR.den --projection-sizex 616 --projection-sizey 480 $PARAMS $OUTPUT
```




## Detector binning
This time I modify the geometry so that I increase number of detector pixels and decrease their lengths produce the following trajectories.

In fact we can imagine that the detector has the dimensions 2464x1920 but in the previous examples we were considerred so called binning. The signals from $4 \times 4$  pixels are put together. Sometimes it is to reduce noise. But sometimes it is from practical reasons that the bandwidth of the device to the reconstructing computer is restricted and this has to be applied.

We will produce camera matrices related to the no binning `CM_CIRCULAR1.den`, $2 \times 2$ binning `CM_CIRCULAR2.den` or $2 \times 2$ binning `CM_CIRCULAR4.den`.

```bash
#Camera matrices relative to the different binning
PARAMS4="--projection-sizex 616 --projection-sizey 480 --pixel-sizex 0.616 --pixel-sizey 0.616"
PARAMS2="--projection-sizex 1232 --projection-sizey 960 --pixel-sizex 0.308 --pixel-sizey 0.308"
PARAMS1="--projection-sizex 2464 --projection-sizey 1920 --pixel-sizex 0.154 --pixel-sizey 0.154"
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS4 CM_CIRCULAR4.den
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS2 CM_CIRCULAR2.den
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS1 CM_CIRCULAR1.den
```
