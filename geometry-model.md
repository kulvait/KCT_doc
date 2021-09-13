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

For the details how to create DEN volume from DICOM series have a look to [this tutorial](link://slug/working-with-kct-cbct-1-converting-dicom-volume-to-den-volume-using-dicom-data-from-public-repository)
