<!--
.. title: Geometry model
.. slug: geometry-model
.. date: 2021-09-13 12:22:25 UTC+02:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

We expect the setup, where the center of the volume of interest is a reference point, normally identified with the zero of the world coordinates. 

In the situation, where user needs a setup, where the zero of word coordinates is not at the center of volume, he or she needs to specify the coordinates of the center of the volume in the world coordinates.

```
    --volume-centerx FLOAT:FLOAT in [-10000 - 10000] Needs: --volume-centery --volume-centerz
                                `X coordinate of the volume center in mm, defaults to 0.00.
    --volume-centery FLOAT:FLOAT in [-10000 - 10000] Needs: --volume-centerx --volume-centerz
                                `Y coordinate of the volume center in mm, defaults to 0.00.
    --volume-centerz FLOAT:FLOAT in [-10000 - 10000] Needs: --volume-centerx --volume-centery
                                `Z coordinate of the volume center in mm, defaults to 0.00.
```
By this mechanism it is possible to reconstruct the volume outside the zero of the world coordinates, that is usually connected with the center of rotation.

The geometry model of the most of the projectors expects that the rotation axis is parallel with the $x_3$ axis of the world coordinates, that the detector cells are rectangle shaped with the dimensions $\chi_1$ and $\chi_2$ and $\chi_2$ is parallel wit the $x_2$.

# Example geometry setup
## Example volume
We use available tools to download and use a publicly available CT volume to demonstrate typical geometry setup and basic work with the tool.
First I selected the dataset from [The Cancer Imaging Archive](https://public.cancerimagingarchive.net/) containing a brain CT scan, ID TCGA-19-1787.
Using the procedure described on (their pages)[https://wiki.cancerimagingarchive.net/display/NBIA/Downloading+TCIA+Images], I have downloaded the set of DICOM files related to this dataset. Because for the purpose of this example, I intend to transform this dataset into the raw format, that can be used by the tool, I would like to mention, that these data are published under the terms of Creative Commons Attribution 3.0 Unported License (http://creativecommons.org/licenses/by/3.0/) and the [following terms](License-to-use-TCGA-GBM-collection-or-derived-patient-data-(Example-CT-scan)). This licencing apply to any user even the user of the derived data, which I publish here in scope of this demonstration.

From the DICOM header we can see that it is a scan on SIEMENS Sensation Cardiac 64 CT machine. It contains 124 slices with the thickness of 1.5mm of the dimension 512x512, the voxel sizes are (0.64453125mm, 0.64453125mm, 1.5mm). To work with such data in the KCT, we first need to convert it to the raw format called DEN that is used by KCT. We do it using [KCT_denpy package](https://github.com/kulvait/KCT_denpy). This package allow us to convert any numpy.ndarray into the DEN file. In DICOM files the attenuations are encoded in Hounsfield units. For the reconstruction or projection of it, it is however needed to convert them into the raw attenuation values. This can be done in the [KCT_dentk](https://github.com/kulvait/KCT_dentk) package by means of the program dentk-fromhu. DEN files created by the described can be found at https://github.com/kulvait/KCT_den_file_opener/releases/download/v1.1.1/ExampleVolumeKCT_TCGA-19-1787.tar.xf, namely DEN_RAW/Series_00.den. To vizualize the volume can be used (KCT Den file opener)[https://github.com/kulvait/KCT_den_file_opener/releases/download/v1.1.1/KCT_Den_File_Opener-1.1.1.jar], a plugin to the (ImageJ)[https://imagej.nih.gov/ij/] viewer that works also with (Fiji)[https://imagej.net/software/fiji/]. It is sufficient to put a file KCT_Den_File_Opener-1.1.1.jar into the .imagej/plugins configuration directory of ImageJ.

Investigating the volume in more detail from the perspective of data alignment it is apparent that the convention described on the picture https://www.slicer.org/wiki/File:Coordinate_sytems.png is observed. So that z axis is the axis of the rotation. Y axis goes from the top (above scanned subject) to bottom (under scanned object). Notice also that ImageJ places the (0,0) top left so that the axial image is vizualized in a natural orientation.


## Example scanning trajectory

Let's reproject the volume we have using theoretical C-Arm FDCT trajectory. Let's say that our device rotates along the axis parallel to the z axis. By angle $\omega$ we denote polar angle between normal to detector that is pointing towards the source and x axis. Let's fix source to isocenter and source to detector lengths and create a setup in which patient is irradiated from 360 different angles corresponding to integer $\omega$ values. Let's create (camera matrices)[https://en.wikipedia.org/wiki/Camera_matrix], which encode this setup. We create them as float64 DEN files, where z dimension will represent number of angles. So that we produce file of dimensions (dimx, dimy, dimz)=(3,4,360).

  

