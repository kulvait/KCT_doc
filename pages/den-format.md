<!--
.. title: DEN format
.. slug: den-format
.. date: 2021-09-13 12:06:01 UTC+02:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

Volume, projection and other types of data are typically stored in the format called DEN, that is format for storing 3D series of the raw little endian values of one of the following types: uint16_t, float32 or float64. There is 6byte header containing three uint16_t numbers representing dimy, dimx, dimz in this order. Then follows a series of the data. The value representing the position (ix,iy,iz) in the volume has flat index $ ix + iy * dimx + iz * dimx * dimy$ 
therefore the data are in row major order.

This format is sometimes also called Dennerlein format, probably after the German CT researcher, Dr.Ing. Frank Dennerlein.

## Extended DEN format

In the original format, the file with the header (0,0,0) is valid as long as it has just 6bytes and no data part. Extended version implemented in KCT is able to represent data with each dimension up to the uint32_t_max. The header in this format is 18bytes and it starts with (0,0,0) or (0,0,1) and continues by (dimy, dimx, dimz) of uint32_t values. Format with initial part of header (0,0,0) has the same row major alignment of the remaining sequence of data, while in the format with initial part of header (0,0,1) the value representing the position (ix,iy,iz) will have a flat index $iy + ix * dimy + iz * dimx * dimy$
therefore the data are in column major order.
