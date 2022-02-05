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

## Legacy DEN

DEN is binary format for storing three dimensional arrays of either uint16, float32 or float64 values. Format has fixed header with size of 6 bytes, which represent three uint16 values corresponding to array dimensions dimy, dimx, dimz in this order. Data are aligned in x-major order and all values are encoded little endian. DEN format or also Dennerlein format, is probably named after the German CT researcher, Dr.Ing. Frank Dennerlein. In the framework is this format used for storing volume, projection and other types of data.

The x-major alignment mean that the value representing the position (ix,iy,iz) in the volume has flat index $ ix + iy * dimx + iz * dimx * dimy$. 

### Problems with legacy format

The type of the data can only be derived from the file size and number of elements represented as given by header. Element size of uint16 is 2 bytes, float32 4 bytes and float64 8 bytes. Thus it is not possible to represent or distinguish two types with 4 bytes element size. Such element must ever be float. This is one limiting factor.

Encoding the dimension along which there is dimension-major alignment on the second position in the header is questionable. It is probably relict from encoding matrices, where we first state number of rows and then number of columns. For multidimensional arrays is this contra-intuitive. Support for y-major alignment is missing. On top of that one dimension might be at most 65535.

Fixing number of dimensions to 3 also reduces flexibility of the format. In some situations would be nice to be able to represent 1D arrays or arrays of greater dimensions than 3D.


## Extended DEN format

We decided to design extended format, which follow a concept of simmilar simplicity as DEN format, however is more flexible, moreover we want these two to be distinguishible from each other in a way that file in extended format can not be valid legacy DEN.

In the original format, the file with the header (0,.,.) is valid as long as it has just 6bytes and no data part. 

Extended version implemented in KCT is able to represent data with each dimension up to the uint32_t_max. The header in this format is 18bytes and it starts with (0,0,0) or (0,0,1) and continues by (dimy, dimx, dimz) of uint32_t values. Format with initial part of header (0,0,0) has the same row major alignment of the remaining sequence of data, while in the format with initial part of header (0,0,1) the value representing the position (ix,iy,iz) will have a flat index $iy + ix * dimy + iz * dimx * dimy$
therefore the data are in column major order.
