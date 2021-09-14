<!--
.. title: KCT framework
.. slug: kct-framework
.. date: 2021-09-13 12:04:57 UTC+02:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

KCT framework is a set of sotware tools to process computer tomography data. It was founded by Vojtěch Kulvait to support his PostDoc research at University of Magdeburg 2018-2021. Work is focused on processing, manipulating and reconstructing the data from the C-Arm CT systems with a flat panel detector, FDCT. It consists of multiple packages described below.

These tools are tested and the compatibility will be maintained towards Debian 11 bullseye with AMD64 CPU. After the compilation, command line programs are built, which use command line parser (CLI11)[https://github.com/CLIUtils/CLI11] to specify inputs, outputs and parameters. BASH scripting is a common practice to put these programs into the CT data processing functional pipelines. 

## [KCT cbct](https://github.com/kulvait/KCT_cbct)

Contains C++ and OpenCL implementation of various methods for algebraic CT reconstruction.

## [KCT dentk](https://github.com/kulvait/KCT_dentk)

It contains tools written in C++ to process files in the [DEN format](DEN-format) to store 3D volume data. Format is derived and compatible with the format reffered as Dennerlein or DEN format.

## [KCT denpy](https://github.com/kulvait/KCT_denpy)

Python package to manipulate filues in DEN format. It uses pydicom package to read DICOM data. Files in formats is possible to convert to numpy.ndarray. By this mechanism is possible to convert series of DICOM files into the DEN files that are used within the framework. 

## [KCT ctiol](https://github.com/kulvait/KCT_ctiol)

C++ library with input output routines for asynchronous thread safe reading/writing CT data. This library is typically a dependency/submodule of other C++ tools in KCT.

## [KCT ctmal](https://github.com/kulvait/KCT_ctmal)

C++ library with mathematic/algebraic algorithms for supporting CT data manipulation. This library is typically a dependency/submodule of other C++ tools in KCT.
