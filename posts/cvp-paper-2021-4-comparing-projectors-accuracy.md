<!--
.. title: CVP paper 2021: Comparing projectors accuracy
.. slug: cvp-paper-2021-4-comparing-projectors-accuracy
.. date: 2021-10-05 22:15:32 UTC+02:00
.. tags: cvp-paper-2021,cvp-paper-2020
.. category: 
.. link: 
.. description: 
.. has_math: true
.. type: text
-->

We have made the following statement in the publication that will be sent for peer review to the [IEEE Transactions on Medical Imaging](https://ieeexplore.ieee.org/xpl/RecentIssue.jsp?punumber=42):

<pre style="white-space: pre-wrap;">
Transparency and reproducibility of the results

We believe in the concept of open science. Therefore, not only the software developed by the first author is published under an open source license, we are also publishing all the procedures, scripts, input data and log files, including implementation details, that were used to produce the graphs and tables in the results section. This way, anyone can reproduce our steps on their hardware and compare the results, or use our logs and scripts to compare the results with the underlying data. The files and protocols are published on the website https://kulvait.github.io/KCT_doc/categories/cvp-paper-2020.html.

This is also important for the users of the software, because in the future we can disclose how the run times of the projectors change with new versions of the software or with new hardware.
</pre>


# Comparison of the accuracy

## Data files
Data files contain projections of the voxels and the differences of the projections with Siddon512 algorithm. Therefore, they are quite large and we split them to the three packages corresponding to individual parts of Figure4. It is `projectorAccuracyComparisonCVPPaper2021_Fig4_top_1x1x5_center.tar.gz`, `projectorAccuracyComparisonCVPPaper2021_Fig4_middle_1x1x1_shifted20-20-20.tar.gz` and `projectorAccuracyComparisonCVPPaper2021_Fig4_bottom_Long.tar.gz`.

These files were split using unix split utility to individual parts, which can be downloaded separatelly from the [this data directory](https://data.stimulate.ovgu.de/d/e413281d4b86410a9f2a/).


Then to extract them the best is to run the following commands
```
cat projectorAccuracyComparisonCVPPaper2021_Fig4_top_1x1x5_center.tar.gz* | tar -xz
cat projectorAccuracyComparisonCVPPaper2021_Fig4_middle_1x1x1_shifted20-20-20.tar.gz* | tar -xz
cat pprojectorAccuracyComparisonCVPPaper2021_Fig4_bottom_Long.tar.gz* | tar -xz
```


## Generating the graphs

To compare accuracy of the projectors and to generate the figure 4 in the paper, we first take a single voxel with attenuation 1. To do so we use `dentk-empty` from [dentk toolbox](https://github.com/kulvait/KCT_dentk).

```
dentk-empty 1 1 1 singleUnitVoxel.den --value 1.0
```
This can be also done using the script 
```
./01createUnitVoxel
```

Next part, which produced the projections and differences is calling the script 
```
./02createProjections
```
It is a bash script to run the reconstructions and create the projections and differences. Beware that to generate Siddon512 might take relatively long even for single voxel.

Finally the graphs can be generated using the command 
```
./03compareProjections.py projectorComparison.csv
```
These commands are present and shall be run in the folders of individual figures.
