<!--
.. title: Working with KCT CBCT 4 First cone beam CT projection
.. slug: working-with-kct-cbct-4-first-cone-beam-ct-projection
.. date: 2021-09-19 14:28:24 UTC+02:00
.. tags: using_kct_blog
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

In the last post Working with KCT CBCT 1 we converted DICOM data to the DEN format and explained how volumes are encoded. Today we create a CT trajectory setup and encode it in the format that can be recognized in KCT. Finally, we use KCT to reproject the volume from the last post using this trajectory.

## Theoretical scanning trajectory

Let's reproject the volume we have using theoretical C-Arm FDCT trajectory. Let's say that our device rotates along the axis parallel to the z axis. By angle $\omega$ we denote polar angle between normal to detector that is pointing towards the source and x axis. Let's fix source to isocenter and source to detector lengths and create a setup in which patient is irradiated from 360 different angles corresponding to integer $\omega$ values. Let's create (camera matrices)[https://en.wikipedia.org/wiki/Camera_matrix], which encode this setup. We create them as float64 DEN files, where z dimension will represent number of angles. So that we produce file of dimensions (dimx, dimy, dimz)=(3,4,360).

## Volume reprojection

Use the following command
```
cllin-projector --cvp --voxel-sizex 0.64453125 --voxel-sizey 0.64453125 --voxel-sizez 1.5 --projection-sizex 616 --projection-sizey 480 DEN_RAW/Series_00.den CM.den Series_00.proj
```
