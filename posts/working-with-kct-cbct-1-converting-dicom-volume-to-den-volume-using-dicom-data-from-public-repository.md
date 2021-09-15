<!--
.. title: Working with KCT CBCT 1 Converting DICOM volume to DEN volume
.. slug: working-with-kct-cbct-1-converting-dicom-volume-to-den-volume-using-dicom-data-from-public-repository
.. date: 2021-09-13 21:05:35 UTC+02:00
.. tags: using_kct_blog
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

In this series of blog posts, I will describe how to work with the KCT framework from the first steps of importing data to the more advanced topics as a running reconstruction. 

In the post today, I explain how to import files in DICOM into the DEN format that is used within the framework. I use the data from the public repository.

## Example dataset

I have choosen a publicly available dataset from [The Cancer Imaging Archive](https://public.cancerimagingarchive.net/) containing a brain CT scan, ID TCGA-19-1787. To download this dataset I followed the procedure described on [The Cancer Imaging Archive page](https://wiki.cancerimagingarchive.net/display/NBIA/Downloading+TCIA+Images). It is needed to instal their application NBIA Data Retriever and provide it with the information in the `manifest-1631446347856.tcia` file, which must contain the following
```
downloadServerUrl=https://public.cancerimagingarchive.net/nbia-download/servlet/DownloadServlet
includeAnnotation=true
noOfrRetry=4
databasketId=manifest-1631446347856.tcia
manifestVersion=3.0
ListOfSeriesToDownload=
1.3.6.1.4.1.14519.5.2.1.5826.4001.312669389023517091391958251391
```

Example dataset is published under the terms of [Creative Commons Attribution 3.0 Unported License](http://creativecommons.org/licenses/by/3.0/) and the [following terms](link://slug/license-tcga-gbm). This licencing apply to any user even the user of the derived data, which I publish here in scope of this demonstration in [Example Volume TCGA-19-1787](https://github.com/kulvait/KCT_den_file_opener/releases/download/v1.1.1/ExampleVolumeKCT_TCGA-19-1787.tar.xf).

When we unpack the data of example volume, we see that dicom files with the suffix dcm are in the directory `TCGA-19-1787/04-20-2002-NA-NR CT HEAD W CE-97133/2.000000-Head Vol.  1.5  H40s ST-51391`. For simplicity I rename this directory just to DICOM and remove parent subdirectories. 
From the DICOM headers we can see that it is a scan on SIEMENS Sensation Cardiac 64 CT machine. It contains $124$ slices with the thickness of $1.5$mm of the dimension $512 \times 512$, the voxel sizes are ($0.64453125$mm, $0.64453125$mm, $1.5$mm).
 
## Conversion of DICOM series to DEN format

For the purpose of this example, we have to transform the DICOM dataset into the [DEN format](link://slug/den-format), which is used by KCT framework. We do it using [KCT_denpy package](https://github.com/kulvait/KCT_denpy). This package allow us to convert between the representations of the volume data as numpy.ndarray, DEN file or DICOM series. Internally it uses the package pydicom to read DICOM data.

For the conversion, I have created a simple script `[DICOMTODEN.py](https://github.com/kulvait/KCT_scripts/blob/master/DICOMTODEN.py)`. The command to convert the data to the DEN format will then be

```bash
DICOMTODEN.py --force --dicom-file-suffix "dcm" DICOM DEN_HU
```

After you run this command, the new directory `DEN_HU` will be created, which contains file `Series_00.den`. This is a converted DICOM file into the DEN format. To vizualize the volume can be used [KCT Den file opener](https://github.com/kulvait/KCT_den_file_opener/releases/download/v1.1.1/KCT_Den_File_Opener-1.1.1.jar), a plugin of [ImageJ](https://imagej.nih.gov/ij/) viewer that works also with [Fiji](https://imagej.net/software/fiji/). It is sufficient to put a file KCT_Den_File_Opener-1.1.1.jar into the .imagej/plugins configuration directory of ImageJ. Then simpy select `File->Open DEN ..` and navigate to `Series_00.den`.

You can notice that the values are integers starting from $-1024$ in the areas of the air up to values greater than $1000$ in the dense bone areas. These are in (Hounsfield units)[https://en.wikipedia.org/wiki/Hounsfield_scale], which are good for the physicans to compare tissue contrasts in a convenient scale, but they do not represent raw attenuation values. To do a projection of the data using KCT or reconstruction of these projections, we need raw attenuation values.

Conversion between Hounsfield units and raw attenuation values is possible using (KCT dentk toolkit)[https://github.com/kulvait/KCT_dentk]. Install the tool `dentk-fromhu` to your path and then the conversion can be done either by calling

```bash
dentk-fromhu Series_00.den Series_00.raw
```
or using the script `DICOMTODEN.py` to internally call `dentk-fromhu` and to produce raw attenuations
```bash
DICOMTODEN.py --force --dicom-file-suffix "dcm" --convert-hu-to-raw --base2 DICOM DEN_RAW
```
There are two caveats. First for the conversion is important the attenuation value of the water, which in Hounsfield units is exactly $0$. By default the conversion will be performed in the same way as with the setting `--water-value 0.027`. If you know the exact attenuation value of the water for given scanner, feel free to change it.

Second there is a problem that in [wiki article about Hounsfield units](https://en.wikipedia.org/wiki/Hounsfield_scale) as well as in virtually every CT textbook, you can find that the minimum Hounsfield unit is 1000. DICOM data from SIEMENS scanners tend to have this minimum value $-1024$. It is due to the better data alignment with uint16 or uint32 representation of offsetted Hounsfield units. Standard conversion formula would however produce negative attenuation values. To correct for this undesired behavior we have to add `--base2` as a parameter either to `DICOMTODEN.py` or `dentk-fromhu` program.

## Vizualizing DEN volume of raw attenuation values derived from DICOM series

When you did everything correct, in the directory `DEN_RAW` is the file `Series_00.den`, which represent raw attenuation values derived from given DICOM series. You can also download [Example Volume TCGA-19-1787](https://github.com/kulvait/KCT_den_file_opener/releases/download/v1.1.1/ExampleVolumeKCT_TCGA-19-1787.tar.xf), which contain also the converted DEN volume.

When vizualizing in ImageJ, you can select an area of the soft tissue and press ALT+CTRL+C. Then you can adjust the contrast. 

Investigating the volume in more detail from the perspective of data alignment it is apparent that the convention described on the picture [CT coordinates, image from the project slicer](https://www.slicer.org/wiki/File:Coordinate_sytems.png) is observed. So that z axis is the axis of the rotation. Y axis goes from the top (above scanned subject) to bottom (under scanned object). Note, that ImageJ places the (0,0) top left and the positive direction of the Y axis is also top to bottom so that the axial image is vizualized in a natural orientation.  

## Working with KCT CBCT 2

In the next post, I will explain what are the conventions for the projection geometry and how to describe the geometric setting of the flat detector CT. We define an example setup of a virtual flat detector CT machine when scannin the volume created in this post. We then use KCT framework to project the volume and create projections.

