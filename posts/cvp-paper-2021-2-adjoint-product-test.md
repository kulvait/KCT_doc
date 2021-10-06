<!--
.. title: CVP paper 2021: Adjoint product test
.. slug: cvp-paper-2021-2-adjoint-product-test
.. date: 2021-10-04 17:29:06 UTC+02:00
.. tags: cvp-paper-2021,cvp-paper-2020
.. category: 
.. link: 
.. description: 
.. type: text
.. has_math: true
-->

We have made the following statement in the publication that will be sent for peer review to the [IEEE Transactions on Medical Imaging](https://ieeexplore.ieee.org/xpl/RecentIssue.jsp?punumber=42):

<pre style="white-space: pre-wrap;">
Transparency and reproducibility of the results

We believe in the concept of open science. Therefore, not only the software developed by the first author is published under an open source license, we are also publishing all the procedures, scripts, input data and log files, including implementation details, that were used to produce the graphs and tables in the results section. This way, anyone can reproduce our steps on their hardware and compare the results, or use our logs and scripts to compare the results with the underlying data. The files and protocols are published on the website https://kulvait.github.io/KCT_doc/categories/cvp-paper-2020.html.

This is also important for the users of the software, because in the future we can disclose how the run times of the projectors change with new versions of the software or with new hardware.
</pre>


Here I show how to perform adjoint product test using KCT CBCT and what are the results on individual platforms.

The problem we are about to solve is as follows. We have a linear operator but we don't have explicit matrix representation of it. Adjoint operator is in real matrices just its transpose. So we have our projector $A$ , backprojector $A^\top$ but how we say that they one is the transpose of another when we could not look to the individual elements? Adjoint product test address this issue, because we know for each vectors $\vec{x}$ $vec{b}$ the following identity shall hold
$$\vec{b} \cdot (A \vec{x}) = \vec{x} \cdot (A^\top \vec{b})$$

When this will hold for a random vectors $\vec{x}$ and $vec{b}$, we can be pretty confident that we have adjoint operators.

In the KCT this adjoint product test is performed in the source file `[adjoint.test.cpp](https://github.com/kulvait/KCT_cbct/blob/master/tests/adjoint.test.cpp)`. The version for what we call in the paper *standard CVP* is implemented in the `TEST_CASE("CVP.AdjointDotProduct.nobarrier_norelaxed_elevationcorrection", "[adjointop][cuttingvox][NOVIZ]")` routine and the version for what we call in the paper *relaxed CVP* is implemented in the `TEST_CASE("CVP.AdjointDotProduct.barrier_relaxed_elevationcorrection", "[adjointop][cuttingvox][NOVIZ]")`. These routines along with others for other parameters of the cutting voxel projector, but also for TT, reported as TA3 projector TEST, and Siddon projector can be run when executing the binary `./test_all -s` to show also the passing tests. We study how far the ratio of the 
$$
\frac{\vec{b} \cdot (A \vec{x})}{\vec{x} \cdot (A^\top \vec{b})} 
$$
is far from optimal value 1.0. The results follow executed with the git commit `09daa3`.

# AMD Radeon VII
```
pci id for fd 5: 1002:66af, driver (null)
pci id for fd 5: 1002:66af, driver (null)
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 2 OpenCL platforms on the PC.
2021-10-04 18:58:35.677 INFO  [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 1: AMD Accelerated Parallel Processing.
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 1.
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 1 OpenCL devices for the platform AMD Accelerated Parallel Processing.
2021-10-04 18:58:35.677 INFO  [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform AMD Accelerated Parallel Processing: gfx906
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/projector.cl
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:58:35.677 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/rescaleProjections.cl
2021-10-04 18:58:35.678 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:102] Building file /tmp/allsources_369982541.cl with options : -Werror
2021-10-04 18:58:36.304 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:58:57.106 INFO  [26327] [____C_A_T_C_H____T_E_S_T____0@:104] Ratio is 1.000000

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
test_all is a Catch v2.11.1 host application.
Run with -? for options

-------------------------------------------------------------------------------
CVP.AdjointDotProduct.nobarrier
-------------------------------------------------------------------------------
/b/KCT/cbct/tests/adjoint.test.cpp:44
...............................................................................

/b/KCT/cbct/tests/adjoint.test.cpp:105: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000071 < 0.00001

2021-10-04 18:58:57.108 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 2 OpenCL platforms on the PC.
2021-10-04 18:58:57.108 INFO  [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 1: AMD Accelerated Parallel Processing.
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 1.
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 1 OpenCL devices for the platform AMD Accelerated Parallel Processing.
2021-10-04 18:58:57.108 INFO  [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform AMD Accelerated Parallel Processing: gfx906
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/projector_cvp_barrier.cl
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:58:57.108 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/rescaleProjections.cl
2021-10-04 18:58:57.108 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:102] Building file /tmp/allsources_127968647.cl with options : -DLOCALARRAYSIZE=7680 -DRELAXED -cl-fast-relaxed-math -Werror
2021-10-04 18:58:57.589 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:59:14.029 INFO  [26327] [____C_A_T_C_H____T_E_S_T____2@:170] Ratio is 1.000000
-------------------------------------------------------------------------------
CVP.AdjointDotProduct.barrier_relaxed
-------------------------------------------------------------------------------
/b/KCT/cbct/tests/adjoint.test.cpp:110
...............................................................................

/b/KCT/cbct/tests/adjoint.test.cpp:171: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.000000041 < 0.00001

2021-10-04 18:59:14.030 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 2 OpenCL platforms on the PC.
2021-10-04 18:59:14.030 INFO  [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 1: AMD Accelerated Parallel Processing.
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 1.
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 1 OpenCL devices for the platform AMD Accelerated Parallel Processing.
2021-10-04 18:59:14.030 INFO  [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform AMD Accelerated Parallel Processing: gfx906
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/projector_cvp_barrier.cl
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:59:14.030 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/rescaleProjections.cl
2021-10-04 18:59:14.031 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:102] Building file /tmp/allsources_336649016.cl with options : -DELEVATIONCORRECTION -DLOCALARRAYSIZE=7680 -DRELAXED -cl-fast-relaxed-math -Werror
2021-10-04 18:59:14.516 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:59:31.450 INFO  [26327] [____C_A_T_C_H____T_E_S_T____4@:236] Ratio is 1.000000
-------------------------------------------------------------------------------
CVP.AdjointDotProduct.barrier_relaxed_elevationcorrection
-------------------------------------------------------------------------------
/b/KCT/cbct/tests/adjoint.test.cpp:176
...............................................................................

/b/KCT/cbct/tests/adjoint.test.cpp:237: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000414 < 0.00001

2021-10-04 18:59:31.451 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 2 OpenCL platforms on the PC.
2021-10-04 18:59:31.451 INFO  [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 1: AMD Accelerated Parallel Processing.
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 1.
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 1 OpenCL devices for the platform AMD Accelerated Parallel Processing.
2021-10-04 18:59:31.451 INFO  [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform AMD Accelerated Parallel Processing: gfx906
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/projector.cl
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:59:31.451 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/rescaleProjections.cl
2021-10-04 18:59:31.452 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:102] Building file /tmp/allsources_529597292.cl with options : -DELEVATIONCORRECTION -Werror
2021-10-04 18:59:32.076 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:59:55.643 INFO  [26327] [____C_A_T_C_H____T_E_S_T____6@:302] Ratio is 1.000000
-------------------------------------------------------------------------------
CVP.AdjointDotProduct.nobarrier_norelaxed_elevationcorrection
-------------------------------------------------------------------------------
/b/KCT/cbct/tests/adjoint.test.cpp:242
...............................................................................

/b/KCT/cbct/tests/adjoint.test.cpp:303: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.00000001 < 0.00001

2021-10-04 18:59:55.644 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 2 OpenCL platforms on the PC.
2021-10-04 18:59:55.644 INFO  [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 1: AMD Accelerated Parallel Processing.
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 1.
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 1 OpenCL devices for the platform AMD Accelerated Parallel Processing.
2021-10-04 18:59:55.644 INFO  [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform AMD Accelerated Parallel Processing: gfx906
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/projector.cl
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/projector_tt.cl
2021-10-04 18:59:55.644 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/backprojector_tt.cl
2021-10-04 18:59:55.645 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:102] Building file /tmp/allsources_1828164295.cl with options : -Werror
2021-10-04 18:59:56.292 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 19:00:09.626 ERROR [26327] [____C_A_T_C_H____T_E_S_T____10@:479] X0.221993, 0.055180, 0.804684
2021-10-04 19:00:09.626 ERROR [26327] [____C_A_T_C_H____T_E_S_T____10@:480] B0.071454, 0.428649, 0.457496
-------------------------------------------------------------------------------
GLSQRReconstructor AdjointDotProduct TA3 projector TEST
-------------------------------------------------------------------------------
/b/KCT/cbct/tests/adjoint.test.cpp:432
...............................................................................

/b/KCT/cbct/tests/adjoint.test.cpp:488: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000331 < 0.00001

2021-10-04 19:00:19.626 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 19:00:19.626 DEBUG [26327] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 19:00:19.626 DEBUG [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 2 OpenCL platforms on the PC.
2021-10-04 19:00:19.626 INFO  [26327] [KCT::util::OpenCLManager::getPlatform@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 1: AMD Accelerated Parallel Processing.
2021-10-04 19:00:19.626 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 1.
2021-10-04 19:00:19.626 DEBUG [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 1 OpenCL devices for the platform AMD Accelerated Parallel Processing.
2021-10-04 19:00:19.626 INFO  [26327] [KCT::util::OpenCLManager::getDevice@/b/KCT/cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform AMD Accelerated Parallel Processing: gfx906
2021-10-04 19:00:19.626 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 19:00:19.626 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 19:00:19.626 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/projector_sidon.cl
2021-10-04 19:00:19.626 DEBUG [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:78] Including file opencl/backprojector_sidon.cl
2021-10-04 19:00:19.626 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:102] Building file /tmp/allsources_966741580.cl with options : -Werror
2021-10-04 19:00:19.850 INFO  [26327] [KCT::Kniha::initializeOpenCL@/b/KCT/cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 19:00:33.125 ERROR [26327] [____C_A_T_C_H____T_E_S_T____14@:664] X0.221993, 0.055180, 0.804684
2021-10-04 19:00:33.126 ERROR [26327] [____C_A_T_C_H____T_E_S_T____14@:665] B0.071454, 0.428649, 0.457496
-------------------------------------------------------------------------------
GLSQRReconstructor AdjointDotProduct Sidon projector TEST
-------------------------------------------------------------------------------
/b/KCT/cbct/tests/adjoint.test.cpp:617
...............................................................................

/b/KCT/cbct/tests/adjoint.test.cpp:673: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000007 < 0.00001

===============================================================================
All tests passed (6 assertions in 7 test cases)
```


# NVIDIA RTX 2080 Ti
```
2021-10-04 18:57:43.924 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:57:43.924 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:57:43.924 DEBUG [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 1 OpenCL platforms on the PC.
2021-10-04 18:57:43.924 INFO  [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 0: NVIDIA CUDA.
2021-10-04 18:57:43.924 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 0.
2021-10-04 18:57:43.924 DEBUG [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 10 OpenCL devices for the platform NVIDIA CUDA.
2021-10-04 18:57:43.924 INFO  [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform NVIDIA CUDA: NVIDIA GeForce RTX 2080 Ti
2021-10-04 18:57:44.099 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:57:44.099 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:57:44.099 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/projector.cl
2021-10-04 18:57:44.099 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:57:44.100 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/rescaleProjections.cl
2021-10-04 18:57:44.100 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:102] Building file /tmp/allsources_1927524877.cl with options : -Werror
2021-10-04 18:57:44.108 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:58:11.721 INFO  [2185335] [____C_A_T_C_H____T_E_S_T____0@:104] Ratio is 1.000000

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
test_all is a Catch v2.11.1 host application.
Run with -? for options

-------------------------------------------------------------------------------
CVP.AdjointDotProduct.nobarrier
-------------------------------------------------------------------------------
/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:44
...............................................................................

/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:105: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000071 < 0.00001

2021-10-04 18:58:11.808 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:58:11.808 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:58:11.808 DEBUG [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 1 OpenCL platforms on the PC.
2021-10-04 18:58:11.808 INFO  [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 0: NVIDIA CUDA.
2021-10-04 18:58:11.808 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 0.
2021-10-04 18:58:11.808 DEBUG [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 10 OpenCL devices for the platform NVIDIA CUDA.
2021-10-04 18:58:11.808 INFO  [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform NVIDIA CUDA: NVIDIA GeForce RTX 2080 Ti
2021-10-04 18:58:11.913 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:58:11.913 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:58:11.913 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/projector_cvp_barrier.cl
2021-10-04 18:58:11.913 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:58:11.913 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/rescaleProjections.cl
2021-10-04 18:58:11.913 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:102] Building file /tmp/allsources_155732126.cl with options : -DLOCALARRAYSIZE=7680 -DRELAXED -cl-fast-relaxed-math -Werror
2021-10-04 18:58:11.917 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:58:18.816 INFO  [2185335] [____C_A_T_C_H____T_E_S_T____2@:170] Ratio is 1.000000
-------------------------------------------------------------------------------
CVP.AdjointDotProduct.barrier_relaxed
-------------------------------------------------------------------------------
/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:110
...............................................................................

/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:171: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000434 < 0.00001

2021-10-04 18:58:18.902 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:58:18.902 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:58:18.902 DEBUG [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 1 OpenCL platforms on the PC.
2021-10-04 18:58:18.902 INFO  [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 0: NVIDIA CUDA.
2021-10-04 18:58:18.902 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 0.
2021-10-04 18:58:18.902 DEBUG [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 10 OpenCL devices for the platform NVIDIA CUDA.
2021-10-04 18:58:18.902 INFO  [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform NVIDIA CUDA: NVIDIA GeForce RTX 2080 Ti
2021-10-04 18:58:19.006 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:58:19.006 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:58:19.006 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/projector_cvp_barrier.cl
2021-10-04 18:58:19.006 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:58:19.006 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/rescaleProjections.cl
2021-10-04 18:58:19.007 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:102] Building file /tmp/allsources_950182500.cl with options : -DELEVATIONCORRECTION -DLOCALARRAYSIZE=7680 -DRELAXED -cl-fast-relaxed-math -Werror
2021-10-04 18:58:19.011 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:58:26.426 INFO  [2185335] [____C_A_T_C_H____T_E_S_T____4@:236] Ratio is 1.000000
-------------------------------------------------------------------------------
CVP.AdjointDotProduct.barrier_relaxed_elevationcorrection
-------------------------------------------------------------------------------
/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:176
...............................................................................

/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:237: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000436 < 0.00001

2021-10-04 18:58:26.511 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:58:26.511 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:58:26.511 DEBUG [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 1 OpenCL platforms on the PC.
2021-10-04 18:58:26.511 INFO  [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 0: NVIDIA CUDA.
2021-10-04 18:58:26.511 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 0.
2021-10-04 18:58:26.511 DEBUG [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 10 OpenCL devices for the platform NVIDIA CUDA.
2021-10-04 18:58:26.511 INFO  [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform NVIDIA CUDA: NVIDIA GeForce RTX 2080 Ti
2021-10-04 18:58:26.621 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:58:26.621 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:58:26.621 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/projector.cl
2021-10-04 18:58:26.621 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:58:26.621 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/rescaleProjections.cl
2021-10-04 18:58:26.621 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:102] Building file /tmp/allsources_2039522998.cl with options : -DELEVATIONCORRECTION -Werror
2021-10-04 18:58:26.625 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:59:12.160 INFO  [2185335] [____C_A_T_C_H____T_E_S_T____6@:302] Ratio is 1.000000
-------------------------------------------------------------------------------
CVP.AdjointDotProduct.nobarrier_norelaxed_elevationcorrection
-------------------------------------------------------------------------------
/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:242
...............................................................................

/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:303: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.00000001 < 0.00001

2021-10-04 18:59:12.251 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 18:59:12.251 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 18:59:12.251 DEBUG [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 1 OpenCL platforms on the PC.
2021-10-04 18:59:12.251 INFO  [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 0: NVIDIA CUDA.
2021-10-04 18:59:12.251 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 0.
2021-10-04 18:59:12.251 DEBUG [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 10 OpenCL devices for the platform NVIDIA CUDA.
2021-10-04 18:59:12.251 INFO  [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform NVIDIA CUDA: NVIDIA GeForce RTX 2080 Ti
2021-10-04 18:59:12.358 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 18:59:12.358 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 18:59:12.358 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/projector.cl
2021-10-04 18:59:12.358 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/backprojector.cl
2021-10-04 18:59:12.358 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/projector_tt.cl
2021-10-04 18:59:12.358 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/backprojector_tt.cl
2021-10-04 18:59:12.359 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:102] Building file /tmp/allsources_934493372.cl with options : -Werror
2021-10-04 18:59:12.363 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 18:59:15.801 ERROR [2185335] [____C_A_T_C_H____T_E_S_T____10@:479] X0.221993, 0.055180, 0.804684
2021-10-04 18:59:15.802 ERROR [2185335] [____C_A_T_C_H____T_E_S_T____10@:480] B0.071454, 0.428649, 0.457496
-------------------------------------------------------------------------------
GLSQRReconstructor AdjointDotProduct TA3 projector TEST
-------------------------------------------------------------------------------
/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:432
...............................................................................

/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:488: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000331 < 0.00001

2021-10-04 19:00:05.821 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:485] projectorLocalNDRange = cl::NDRange()
2021-10-04 19:00:05.821 DEBUG [2185335] [KCT::BaseReconstructor::BaseReconstructor@:497] backprojectorLocalNDRange = cl::NDRange(4, 16, 1)
2021-10-04 19:00:05.821 DEBUG [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:30] There exists 1 OpenCL platforms on the PC.
2021-10-04 19:00:05.821 INFO  [2185335] [KCT::util::OpenCLManager::getPlatform@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:32] Selected OpenCL platform 0: NVIDIA CUDA.
2021-10-04 19:00:05.821 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:48] Adding deviceID 0 on the platform 0.
2021-10-04 19:00:05.821 DEBUG [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:81] There exists 10 OpenCL devices for the platform NVIDIA CUDA.
2021-10-04 19:00:05.821 INFO  [2185335] [KCT::util::OpenCLManager::getDevice@/home/kulvait/git/kct_cbct/submodules/CTIOL/src/OPENCL/OpenCLManager.cpp:83] Selected device 0 on the platform NVIDIA CUDA: NVIDIA GeForce RTX 2080 Ti
2021-10-04 19:00:05.929 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/utils.cl
2021-10-04 19:00:05.929 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/include.cl
2021-10-04 19:00:05.929 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/projector_sidon.cl
2021-10-04 19:00:05.929 DEBUG [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:78] Including file opencl/backprojector_sidon.cl
2021-10-04 19:00:05.929 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:102] Building file /tmp/allsources_2000313764.cl with options : -Werror
2021-10-04 19:00:05.932 INFO  [2185335] [KCT::Kniha::initializeOpenCL@/home/kulvait/git/kct_cbct/src/Kniha.cpp:152] Build succesfull
2021-10-04 19:00:09.358 ERROR [2185335] [____C_A_T_C_H____T_E_S_T____14@:664] X0.221993, 0.055180, 0.804684
2021-10-04 19:00:09.358 ERROR [2185335] [____C_A_T_C_H____T_E_S_T____14@:665] B0.071454, 0.428649, 0.457496
-------------------------------------------------------------------------------
GLSQRReconstructor AdjointDotProduct Sidon projector TEST
-------------------------------------------------------------------------------
/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:617
...............................................................................

/home/kulvait/git/kct_cbct/tests/adjoint.test.cpp:673: PASSED:
  REQUIRE( std::abs(adjointProductRatio - 1.0) < tol )
with expansion:
  0.0000000007 < 0.00001

===============================================================================
All tests passed (6 assertions in 7 test cases)
```

