<!--
.. title: Synchrotron data analysis 1: Flat field correction
.. slug: synchrotron-data-analysis-1-flat-field-correction
.. date: 2022-08-25 21:05:35 UTC+02:00
.. tags: synchrotron_blog
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
.. status: private
-->

My recent work deals with the analysis of synchrotron data for computed tomography. In this series of blog posts I intend to describe some important concepts of the analysis and the tools that can be used for dealing with related issues.

First step when dealing with the synchrotron data is processing of the raw projection data. Typically it consists from pixel binning, current normalization and flat field correction. In this post I will explain how to use KCT for flat field correction. 

## Input data

To do standard flat field correction three sets of projection data are necessary. Let's say MxN are dimensions of the detector, then we need the following three datasets:


| Field      | X-rays | File     | Dimensions | Explanation                                                                                                                      |
|------------|--------|----------|------------|----------------------------------------------------------------------------------------------------------------------------------|
| Image data | on     | img.den  | MxNx$n_img$  | Intensity data obtained from the detector when scanning object                                                                   |
| Flat field | on     | flat.den | MxNx$n_flat$ | Intensity data obtained from the detector without scanning object present, other parameters of the imaging system being the same |
| Dark field | off    | dark.den | MxNx$n_dark$ | Intensity data obtained from the detector without scanning object present without incomming X-rays                               | 


### Standard flat field correction
With the scalar quantity G the formula for the flat field correction is as follows
`IMG_cor = G * (IMG - DARK)/(FLAT - DARK)` 
Standard flat field correction of the image is pixel-wise, where we first average flat field and dark field to obtain.


`dentk-framecalc1 --avg flat.den flat_avg.den`
`dentk-framecalc1 --avg dark.den dark_avg.den`
Then we subtract dark from flat
`dentk-calc --subtract flat_avg.den dark_avg.den flat_minus_dark.den`
Then we want to subtract dark_avg.den from every img.den frame.
`dentk-framecalc --subtract img.den dark_avg.den img_minus_dark.den`
We can now compute the extinction using
`dentk-framecalc --divide img_minus_dark.den flat_minus_dark.den cor.den`
`dentk-calc1 --inv cor.den expext.den`
`dentk-calc1 --log expext.den ext.den`




