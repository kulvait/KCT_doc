<!--
.. title: Perfusion analysis 1 Reprojected perfusion CBCT scan to DICOM
.. slug: perfusion-analysis-1-reprojected-perfusion-cbct-scan-to-dicom
.. date: 2023-01-12 16:05:36 UTC+01:00
.. tags: perfusion_analysis_blog
.. category: 
.. link: 
.. description: 
.. type: text
.. status: private
-->

CBCT perfusion data can be stored in many different formats. For data scientists the DICOM format is usually not the most convenient so the first think I do is usually to store image data in some other format and add the description of the acquisition that I need for processing. This approach has worked so far but recently I am facing inverse problem. I have produced reprojected CBCT data in scope of [my article](https://doi.org/10.1002/mp.15640). Now to share such datasets with medical people is a challenging, because virtually every tool they use is expecting input to be in DICOM format. Therefore in this post I describe how we can export series of 10 volumes, which were artificially created to represent simulated CBCT data into the DICOM format in order to process them by the tools, which expect DICOM output.

It is not hard to say why the "IT people" in medicine do not want to bother with DICOM format. [DICOM standard](https://www.dicomstandard.org/current) has 22 parts and virtually every of them has well over 100 pages. So my ambition is not to describe how to produce 100% correct perfusion DICOM data, but rather to export necessary parts which allow software for perfusion processing to read such dataset.


