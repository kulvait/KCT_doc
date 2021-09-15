<!--
.. title: KCT internals 1 comparing projectors
.. slug: kct-internals-1-comparing-projectors
.. date: 2021-09-20 09:14:20 UTC+02:00
.. tags: kct_internals
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

In this series of the blog posts I will go into the details of the KCT package that actually do the computations. I would like to convince you that it is also important not to know only interface but how the things work inside as well.

Today I will discuss the issue how to compare different projectors. I have implemented cutting voxel projector (CVP), I think that it is a very nicely structured projector that exploit correspondence between volume integrals of the attenuation and projections. At the end of the day, I have to compare my projector to the established projectors. And how to do it when every projector is original?

I start with the algorithm due to Siddon [Siddon projector](https://doi.org/10.1118/1.595715) from 1984, which is computing exact paths through the voxelized objects. It is interesting that noone was able to came forward with better algorithm then. There are simpler ray casting approaches, but they are more inexact and tend to be slower. This is the only projector in the package that computes attenuation along the rays. Other projectors are trying to somehow estimate what happens, when we cast infinite number of the rays towards each detector pixel.

Let's first priepare camera matrices for the setup analogous to [CBCT geometry post](link:://slug//working-with-kct-cbct-2-projective-geometry-and-camera-matrices-to-describe-ct-geometry) but let's specify slightly different parameters.

