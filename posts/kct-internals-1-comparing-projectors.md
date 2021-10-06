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

In this series of the blog posts I will go into the details of the KCT package, which actually do the computations. I would like to convince you that it is also important not to know only interface but how the things work inside as well.

Today I will discuss the issue how to compare different projectors. 
[//]: # (I have implemented cutting voxel projector (CVP), I think that it is a very nicely structured projector that exploit correspondence between volume integrals of the attenuation and projections. To convince other people, I need to compare my projector to the established projectors. And how to do it when every projector is slightly different?)

We will use well established class of projectors, so called ray tracers. In my opinion the best ray tracer is that based on the algorithm due to Siddon [Siddon projector](https://doi.org/10.1118/1.595715) from 1984, which is computing exact paths through the voxelized objects. The algorithm is so simple and briliant, that noone else has come forward with something faster. There are simpler ray casting approaches, but they are more inexact and when trying to reach some level of exactness for example by means of increasing the increment step, they tend to be slower. Therefore in KCT, I decided to implement Siddon algorithm for ray casting in order to have the projector that computes attenuation along the rays in the package. Other projectors implamented are trying to somehow estimate what happens, when we cast infinite number of the rays towards each detector pixel.


## Detector binning
Detector binning is common technique for flat panel detector. For example in $4 \times 4$ binning the adjacent goups of  $4 \times 4$ pixels are considered as one big pixel. This technique can be used to reduce the noise or reduce the amount of data transferred from the device to the computer.

Let's first priepare camera matrices for the setup analogous to [CBCT geometry post](link:://slug//working-with-kct-cbct-2-projective-geometry-and-camera-matrices-to-describe-ct-geometry) and [Python implementation post](working-with-kct-cbct-3-python-implementation-of-circular-ct-trajectory). This time let's expect detector binning 4x4 was used.


```bash
mkdir -p geometry
PARAMS="--projection-sizex 616 --projection-sizey 480 --pixel-sizex 0.616 --pixel-sizey 0.616"
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS geometry/CM4x4_616x480_CIRCULAR.den
```
We can then decrease the size of pixels to produce matrices with different binnings. Note that when we keep the number of pixels constant, the detector size will decrease, but for the following demonstration this is not undesirable.
```bash
#Camera matrices relative to the different binning and different numbers of voxels on the detector
#2x2 binning
PARAMS="--projection-sizex 1232 --projection-sizey 960 --pixel-sizex 0.308 --pixel-sizey 0.308"
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS geometry/CM2x2_1232x960_CIRCULAR.den
PARAMS="--projection-sizex 616 --projection-sizey 480 --pixel-sizex 0.308 --pixel-sizey 0.308"
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS geometry/CM2x2_616x480_CIRCULAR.den
#1x1 binning
PARAMS="--projection-sizex 2464 --projection-sizey 1920 --pixel-sizex 0.154 --pixel-sizey 0.154"
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS geometry/CM1x1_2464x1920_CIRCULAR.den
PARAMS="--projection-sizex 1232 --projection-sizey 960 --pixel-sizex 0.154 --pixel-sizey 0.154"
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS geometry/CM1x1_1232x960_CIRCULAR.den
PARAMS="--projection-sizex 616 --projection-sizey 480 --pixel-sizex 0.154 --pixel-sizey 0.154"
createCameraMatricesForCircularScanTrajectory.py --force --write-params-file $PARAMS geometry/CM1x1_616x480_CIRCULAR.den
```

## Comparing projectors
We can compare different projectors by comparing the way how they project a single voxel on the projector during circular trajectory scan. Let's make the volume containing just single voxel with the attenuation value $1$. 
```bash
dentk-empty 1 1 1 singleUnitVoxel.den --value 1.0
```
Let's suppose the voxel is centerred at $(0,0,0)$ and its dimensions are $1mm \times 1mm \times 1mm$. 
Next, create the directory for working with this voxel and for projections to keep our working directory cleaner and export parameters we will be working with
```bash
mkdir -p 1x1x1_center
cd 1x1x1_center
ln -s ../geometry/ geometry
mkdir -p projections
export CM="geometry/CM1x1_616x480_CIRCULAR.den"
export VPARAMS="--voxel-sizex 1.0 --voxel-sizey 1.0 --voxel-sizez 1.0"
```
Let's do an initial testing projection using Siddon projector

```bash
PARAMS="--probes-per-edge 1 --siddon"
OUTPUT="projections/voxel.siddon1"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
```
When we visualize the file `projections/voxel.siddon1` in ImageJ, we can see the dot in the center of the projector surface that corresponds well with our geometric intuition.
Casting one ray towards each pixel is a low number for actual comparison of the projectors. Therefore let's cast $512x512$ rays to get a baseline.
```bash
CM="geometry/CM1x1_616x480_CIRCULAR.den"
VPARAMS="--voxel-sizex 1.0 --voxel-sizey 1.0 --voxel-sizez 1.0"
PARAMS="--probes-per-edge 512 --siddon"
OUTPUT="projections/voxel.siddon512"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
```
You can see the projection gets slower. Its because towards each detector pixel is now casted $512 \times 512 = 262.144$ rays. And there is $616 \times 480$ pixels and $360$ views. So in total around 28 trilions (12 zeros, so that ~28,000,000,000,000) rays are casted. I'd like to mention that in `cllin-projector` the running time of the projections is slower than the running time of the projection in the reconstruction program `cllin-krylov`. The resason is that we typically need this for debugging or one time reprojection and it is written to allow me faster debugging. The place, where the projectors and backprojectors are optimized for the speed is the reconstruction by means of `cllin-krylov`.

Now we have our very fine approximation of the actual shape of the voxel projection `voxel.sidon512`. However how do we compare it to `voxel.sidon1`? First make a difference 
```bash
dentk-calc --force --subtract projections/voxel.siddon1 projections/voxel.siddon512 voxel.siddon1_minus_siddon512
```

Of course we can conpute a L2 norm or RMSE of the file `voxel.siddon1_minus_siddon512`
```bash
dentk-info --l2norm voxel.siddon1_minus_siddon512
#17.1
```
We can see that there are surely differences between projectors when the number of casted rays is higher. Let's systematically compare projectors for 1mm center voxels and compute view based statistics.
```bash
export CM="geometry/CM1x1_616x480_CIRCULAR.den"
export VPARAMS="--voxel-sizex 1.0 --voxel-sizey 1.0 --voxel-sizez 1.0"
PARAMS="--probes-per-edge 512 --siddon"
OUTPUT="projections/voxel.siddon512"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 256 --siddon"
OUTPUT="projections/voxel.siddon256"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 128 --siddon"
OUTPUT="projections/voxel.siddon128"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 64 --siddon"
OUTPUT="projections/voxel.siddon64"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 32 --siddon"
OUTPUT="projections/voxel.siddon32"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 16 --siddon"
OUTPUT="projections/voxel.siddon16"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 8 --siddon"
OUTPUT="projections/voxel.siddon8"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 4 --siddon"
OUTPUT="projections/voxel.siddon4"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 2 --siddon"
OUTPUT="projections/voxel.siddon2"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--probes-per-edge 1 --siddon"
OUTPUT="projections/voxel.siddon1"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--cvp"
OUTPUT="projections/voxel.cvp"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--cvp --barrier --relaxed"
OUTPUT="projections/voxel.cvb"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
PARAMS="--tt"
OUTPUT="projections/voxel.tt"
cllin-projector --force $VPARAMS $PARAMS singleUnitVoxel.den $CM --projection-sizex 616 --projection-sizey 480 $OUTPUT
```

```bash
dentk-calc --force --subtract projections/voxel.siddon1 projections/voxel.siddon100 voxel.siddon1_minus_siddon100
dentk-calc --force --subtract projections/voxel.siddon50 projections/voxel.siddon100 voxel.siddon50_minus_siddon100
dentk-calc --force --subtract projections/voxel.cvp projections/voxel.siddon100 voxel.cvp_minus_siddon100
dentk-calc --force --subtract projections/voxel.cvb projections/voxel.siddon100 voxel.cvb_minus_siddon100
dentk-calc --force --subtract projections/voxel.tt projections/voxel.siddon100 voxel.tt_minus_siddon100
#siddon100_ctttk
dentk-calc --force --subtract projections/voxel.siddon1 projections/voxel.siddon100_ctttk voxel.siddon1_minus_siddon100_ctttk
dentk-calc --force --subtract projections/voxel.siddon50 projections/voxel.siddon100_ctttk voxel.siddon50_minus_siddon100_ctttk
dentk-calc --force --subtract projections/voxel.siddon100 projections/voxel.siddon100_ctttk voxel.siddon100_minus_siddon100_ctttk
dentk-calc --force --subtract projections/voxel.siddon512 projections/voxel.siddon100_ctttk voxel.siddon512_minus_siddon100_ctttk
dentk-calc --force --subtract projections/voxel.cvp projections/voxel.siddon100_ctttk voxel.cvp_minus_siddon100_ctttk
dentk-calc --force --subtract projections/voxel.cvb projections/voxel.siddon100_ctttk voxel.cvb_minus_siddon100_ctttk
dentk-calc --force --subtract projections/voxel.tt projections/voxel.siddon100_ctttk voxel.tt_minus_siddon100_ctttk
#siddon512
dentk-calc --force --subtract projections/voxel.siddon128 projections/voxel.siddon512 voxel.siddon128_minus_siddon512
dentk-calc --force --subtract projections/voxel.siddon64 projections/voxel.siddon512 voxel.siddon64_minus_siddon512
dentk-calc --force --subtract projections/voxel.siddon1 projections/voxel.siddon512 voxel.siddon1_minus_siddon512
dentk-calc --force --subtract projections/voxel.cvp projections/voxel.siddon512 voxel.cvp_minus_siddon512
dentk-calc --force --subtract projections/voxel.cvb projections/voxel.siddon512 voxel.cvb_minus_siddon512
dentk-calc --force --subtract projections/voxel.tt projections/voxel.siddon512 voxel.tt_minus_siddon512
```
Results
```bash
#--voxel-sizex 1.0 --voxel-sizey 1.0 --voxel-sizez 1.0
dentk-info --l2norm single1Voxel.siddon1_minus_siddon100
#17.0 RMSE 0.001651
dentk-info --l2norm single1Voxel.tt_minus_siddon100
#0.1 RMSE=0.000009
dentk-info --l2norm single1Voxel.cvp_minus_siddon100
#0.1 RMSE=0.000009
dentk-info --l2norm single1Voxel.cvb_minus_siddon100
#0.1 RMSE=0.000010
```
```bash
#--voxel-sizex 1.0 --voxel-sizey 1.0 --voxel-sizez 5.0
dentk-info --l2norm single1Voxel.siddon1_minus_siddon100
#13.0
dentk-info --l2norm single1Voxel.tt_minus_siddon100
#0.1 RMSE=0.000014
dentk-info --l2norm single1Voxel.cvp_minus_siddon100
#0.1 RMSE=0.000006
dentk-info --l2norm single1Voxel.cvb_minus_siddon100
#0.1 RMSE=0.000009
```
To get more complete statistic just run
```
dentk-info --l2norm single1Voxel.siddon1_minus_siddon100 --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.siddon1_minus_siddon100.log
dentk-info --l2norm single1Voxel.cvb_minus_siddon100 --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.cvb_minus_siddon100.log
dentk-info --l2norm single1Voxel.cvp_minus_siddon100 --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.cvp_minus_siddon100.log
dentk-info --l2norm single1Voxel.tt_minus_siddon100 --frames 0-end | grep RMSE | grep frame |awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.tt_minus_siddon100.log
echo "Angle" > angles
dentk-info --l2norm single1Voxel.siddon1_minus_siddon100 --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $4}' >> angles
echo "Baseline" > siddon.100
echo "CVP" > cvp.l2
echo "CVB" > cvb.l2
echo "TT" > tt.l2
echo "Siddon1" > siddon.l2
dentk-info --l2norm projections/single1Voxel.siddon100 --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> siddon.100
dentk-info --l2norm single1Voxel.siddon1_minus_siddon100 --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> siddon.l2
dentk-info --l2norm single1Voxel.cvb_minus_siddon100 --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> cvb.l2
dentk-info --l2norm single1Voxel.cvp_minus_siddon100 --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> cvp.l2
dentk-info --l2norm single1Voxel.tt_minus_siddon100 --frames 0-end | grep RMSE | grep frame |awk -F'[|, ]' '{print $9 }' >> tt.l2
paste angles siddon.100 cvp.l2 cvb.l2 tt.l2 siddon.l2 > projectorComparison.csv
```

```
dentk-info --l2norm single1Voxel.siddon1_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.siddon1_minus_siddon100_ctttk.log
dentk-info --l2norm single1Voxel.cvb_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.cvb_minus_siddon100_ctttk.log
dentk-info --l2norm single1Voxel.cvp_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.cvp_minus_siddon100_ctttk.log
dentk-info --l2norm single1Voxel.tt_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame |awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.tt_minus_siddon100_ctttk.log
dentk-info --l2norm single1Voxel.siddon100_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame |awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.siddon100_minus_siddon100_ctttk.log
dentk-info --l2norm single1Voxel.siddon512_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame |awk -F'[|, ]' '{print $4 "\t" $9 "\t" $11 }' > single1Voxel.siddon512_minus_siddon100_ctttk.log
echo "Angle" > angles
dentk-info --l2norm single1Voxel.siddon1_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $4}' >> angles
echo "Baseline" > siddonCTL.100
echo "Siddon1" > siddon1.l2
echo "Siddon100" > siddon100.l2
echo "Siddon512" > siddon512.l2
echo "CVP" > cvp.l2
echo "CVB" > cvb.l2
echo "TT" > tt.l2
dentk-info --l2norm projections/single1Voxel.siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> siddonCTL.100
dentk-info --l2norm single1Voxel.siddon1_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> siddon1.l2
dentk-info --l2norm single1Voxel.siddon100_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> siddon100.l2
dentk-info --l2norm single1Voxel.siddon512_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> siddon512.l2
dentk-info --l2norm single1Voxel.cvb_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> cvb.l2
dentk-info --l2norm single1Voxel.cvp_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame | awk -F'[|, ]' '{print $9 }' >> cvp.l2
dentk-info --l2norm single1Voxel.tt_minus_siddon100_ctttk --frames 0-end | grep RMSE | grep frame |awk -F'[|, ]' '{print $9 }' >> tt.l2
paste angles siddonCTL.100 siddon1.l2 siddon100.l2 siddon512.l2 cvp.l2 cvb.l2 tt.l2 > projectorComparisonCTL.csv
```



Now we see an interesting object, voxel of the cube with $10cm$ edge. Here are now seen even some cone beam effect on the projection data as can be shown in ImageJ.
Let's say now that we shall cast more rays towards each pixel, let's say it will be 100 to have a good approximation 


but we get just some numbers without direct relevance. Better Approach would be to compare L2 norm of the difference to the L2 norm of our ground truth data represented by `single100Voxel.sidon100`.
```bash
dentk-info --l2norm projections/single100Voxel.siddon100
```
Now from the fraction of L2 norms we can tell something about relative norm of error
```
805.5/475233.8=0.0017
```
so there is a difference arount 2 promiles between the detectors. Now we would like to get better information. So let's compare L2 norms of individual views and plot it into the graph.


