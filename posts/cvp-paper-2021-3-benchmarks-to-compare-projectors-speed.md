<!--
.. title: CVP paper 2021: Benchmarks to compare projectors speed
.. slug: cvp-paper-2021-3-benchmarks-to-compare-projectors-speed
.. date: 2021-10-05 16:06:54 UTC+02:00
.. tags: cvp-paper-2021,cvp-paper-2020
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

We have made the following statement in the publication that will be sent for peer review to the [IEEE Transactions on Medical Imaging](https://ieeexplore.ieee.org/xpl/RecentIssue.jsp?punumber=42):

<pre style="white-space: pre-wrap;">
Transparency and reproducibility of the results

We believe in the concept of open science. Therefore, not only the software developed by the first author is published under an open source license, we are also publishing all the procedures, scripts, input data and log files, including implementation details, that were used to produce the graphs and tables in the results section. This way, anyone can reproduce our steps on their hardware and compare the results, or use our logs and scripts to compare the results with the underlying data. The files and protocols are published on the website https://kulvait.github.io/KCT_doc/categories/cvp-paper-2020.html.

This is also important for the users of the software, because in the future we can disclose how the run times of the projectors change with new versions of the software or with new hardware.
</pre>


# Comparion of the speed

Here we describe precisely how we perform comparsion of the projector and backprojector speeds.

Link to the log files of the speed comparison can be found [here](https://data.stimulate.ovgu.de/f/be84e799fd774deabeb5/?dl=1).

From the tests we performed were prepared the following deterministic benchmarks. The tests can be run with the reproducible setup using precisely defined [input data](https://data.stimulate.ovgu.de/f/be84e799fd774deabeb5/?dl=1), camera matrices representing geometries and projection data. Aim of both benchmarks is to run an itterative reconstruction process and precisely measure the speeds of projectors and bacprojector during it. The data reconstructed is deterministically generated noise from uniform distribution [0,1]. These data are disclosed for each benchmark.

For given setup 40 itterations of an CT reconstruction technique shall be used, which performs exactly 40 projections and 40 backprojections. After the initialization, that might be after the data are loaded into the GPU memmory, before the first backprojection start, starts the time measurement. Between the backprojection and consecutive projection is reported the difference time as the time of backprojection. It is not possible to exclude time between backprojection and consecutive projection from the measurement. Exact placement of the measured point is on the implementation. Time can be stopped after the last projection is executed, before writing the data to the disk. Then the average time of projection and backprojection is computed.

In our tests we used CGLS, because it does not perform intensive operations but projections and backprojections. 

The tests use the `createCameraMatricesForCircularScanTrajectory.py` script for creating geometry setup of circular scan. Details of the implementation and derivation can be found in this [blog post](link://slug/working-with-kct-cbct-2-projective-geometry-and-camera-matrices-to-describe-ct-geometry).

## Benchmark YL

Based on the setup from the paper http://doi.org/10.1109/TMI.2010.2050898

Data to reconstruct were prepared using the command `dentk-empty` from [dentk toolbox](https://github.com/kulvait/KCT_dentk). It simulates 720 views of the $512 \times 512$ detector matrix.
```
dentk-empty --noise 512 512 720 noise512x512x720.den
```
Even through the current implementation of `dentk-empty` shall produce deterministic noise, we include the file `noise512x512x720.den` into the [data disclosed](https://data.stimulate.ovgu.de/f/be84e799fd774deabeb5/?dl=1).

The camera matrices, which define given geometry was created using the command `createCameraMatricesForCircularScanTrajectory.py` from [scripts repository](https://github.com/kulvait/KCT_scripts).
```
createCameraMatricesForCircularScanTrajectory.py --write-params-file --projection-sizex 512 --projection-sizey 512 --pixel-sizex 1.0 --pixel-sizey 1.0 --source-to-detector 949 --source-to-isocenter 541 --number-of-angles 720 YLBenchmarkInputData/CMLong720_512x512.den
```
These input files define the input setup of the test. 

## Benchmark TP
Based on the setup from T.  Pfeiffer,  R.  Frysch,  and  G.  Rose,  “Two  extensions  of  the  separablefootprint  forward  projector,”  16th International Meeting on FullyThree-Dimensional Image Reconstruction in Radiology and NuclearMedicine, 2021.

Data to reconstruct were prepared using the command `dentk-empty` from [dentk toolbox](https://github.com/kulvait/KCT_dentk). It simulates 100 views of the $1280 \times 960$ detector matrix.

```
dentk-empty --noise 1280 960 100 noise1280x960x100.den
```

Even through the current implementation of `dentk-empty` shall produce deterministic noise, we include the file `noise1280x960x100.den` into the [data disclosed](https://data.stimulate.ovgu.de/f/be84e799fd774deabeb5/?dl=1).

The camera matrices, which define given geometry was created using the command `createCameraMatricesForCircularScanTrajectory.py` from [scripts repository](https://github.com/kulvait/KCT_scripts).
```
createCameraMatricesForCircularScanTrajectory.py --write-params-file --projection-sizex 1280 --projection-sizey 960 --pixel-sizex 0.25 --pixel-sizey 0.25 --omega-zero 0 --omega-angular-range 198 --source-to-detector 1000 --source-to-isocenter 750 --number-of-angles 100 shortScanTP100_1280x960.den
```
These input files define the input setup of the test. 



# Results of the benchmarks for CVP paper

Log files and input files are disclosed [here](https://data.stimulate.ovgu.de/f/be84e799fd774deabeb5/?dl=1).
The [results logs](https://data.stimulate.ovgu.de/f/be84e799fd774deabeb5/?dl=1) can be parsed and the average projection and backprojection times obtained by running the script
```
averageTimes log_file
```
from the https://github.com/kulvait/KCT_scripts/tree/master/reconstruction

The structure of the data is as follows:

In the speedComparison folders there are the log files from which the resulting speeds of corresponding projectors/backprojectors were generated, commit of KCT and the full command is also disclosed in each log file. Particular input projection matrices and projection data can be found at TPBenchmarkInputData and YLBenchmarkInputData.

YL=Table I from the paper, TP= Table II from the paper

Logs are from several CVP projector/backprojector settings. Only few are refered in the tables of the paper. There is the mapping of projectors/backprojectors to the log files:

CVP=nobar_norel_elv.log

CVP relaxed=bar_rel_elv.log

TT=tt.log

Siddon8=siddon8.log

