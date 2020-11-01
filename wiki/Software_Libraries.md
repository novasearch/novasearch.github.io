---
title: Configuring your home (Libraries and Frameworks)
permalink: wiki/Software_Libraries/
layout: wiki
---

Some guidelines/tutorials for installing some common libraries and frameworks in the Rocks Cluster (Work in Progress).


FFmpeg
------------

Steps for installing FFmpeg [link](https://ffmpeg.org/download.html).

Follow this tutorial:
[Link](https://trac.ffmpeg.org/wiki/CompilationGuide/Ubuntu). The instructions below introduce some modifications required for correct compilation on the Rocks cluster and for enabling linkage with OpenCV, thus they should be taken into account. To link later with OpenCV it is important to compile all ffmpeg dependencies as shared libraries. This is usually accomplished by passing the option '--enable-shared' and '--enable-pic' to the configure script. If you get any error when compiling any of the dependencies saying that a specific flag is not recognized (e.g. --enable-pic) just remove it from the command.

Configure command to compile YASM:

    $ ./configure --prefix="$HOME/<build folder>" --bindir="$HOME/<bins folder>" --enable-pic --enable-shared

Configure command to compile x264:

    $ PATH="$HOME/<bins folder>:$PATH" ./configure --prefix="$HOME/<build folder>" --bindir="$HOME/<bins folder>" --enable-static --disable-opencl --enable-pic --enable-shared

Configure command to compile x265:

    $ PATH="$HOME/<bins folder>:$PATH" cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="$HOME/<build folder>"  -DENABLE_SHARED:bool=on ../../source

Configure command to compile libfdk-aac:

    $ ./configure --prefix="$HOME/<build folder>" --enable-shared --enable-pic

Configure command to compile libmp3lame:

    $ LIBS=-ltinfo ./configure --prefix="$HOME/<build folder>" --enable-nasm --enable-shared --enable-pic

Configure command to compile libopus:

    $ ./configure --enable-shared --enable-pic

The configure of FFmpeg will complain about libass. Then libass will complain fribidi and freetype2. Download, compile and install fribidi and then libass libraries. Remember to set the --prefix of every library to the build folder. For freetype2, we only need to export to PKG\_CONFIG\_PATH variable the freetype2.pc file:

    $ export PKG_CONFIG_PATH=/usr/lib64/pkgconfig/:$PKG_CONFIG_PATH

Pass the flag '--disable-require-system-font-provider' to the libass configure command.

Then it will complain about libtheora [link](https://www.theora.org/downloads/). Download libtheora from the provided link. When compiling libtheora take into account the dependencies mentioned in the website and compile it as shared library.

Configure command to compile FFmpeg:

    $ PATH="$HOME/bin:$PATH" PKG_CONFIG_PATH="$HOME/<build folder>/lib/pkgconfig/":$PKG_CONFIG_PATH ./configure   --prefix="$HOME/<build folder>"   --pkg-config-flags="--static"   --extra-cflags="-I$HOME/<build folder>/include"   --extra-ldflags="-L$HOME/<build folder>/lib"   --bindir="$HOME/<bins folder>"   --enable-gpl   --enable-libass   --enable-libfdk-aac   --enable-libfreetype   --enable-libmp3lame   --enable-libopus   --enable-libtheora   --enable-libvorbis   --enable-libvpx   --enable-libx264   --enable-libx265 --enable-nonfree --enable-avresample --enable-shared


OpenCV
------

Steps for installing OpenCV 3.2.0 with extra modules (OpenCV contrib).
OpenCV will be compiled with support for OpenCL, FFmpeg, Python 2.7 and
3.5 (both from anaconda).

Downloading OpenCV 3.2.0: [URL](https://github.com/opencv/opencv/releases/tag/3.2.0). Release 3.2.0 of Extra modules must be downloaded: [URL](https://github.com/opencv/opencv_contrib/releases/tag/3.2.0).

NOTE: MAKE SURE THAT OpenCV and OpenCV-Contrib VERSIONS ARE THE SAME!

From now on, let <opencv_contrib_dir> be the the downloaded <opencv\_contrib> folder.

Unload EVERYTHING and load ONLY the necessary modules:

    $ module purge  
    $ module load cmake gnu gnutools eigen mvapich2_eth hdf5

Make sure module cuda and mkl are not loaded.

GCC must know where the file mpi.h is. When the mvapich2\_eth module is loaded the environment variable CPATH is not updated. Update the CPATH variable:

    $ export CPATH=/opt/mvapich2/gnu/eth/include/:$CPATH

Compiling OpenCV:

    $ cd <opencv_source>  
    $ mkdir build && cd "$_"
    $ CXXFLAGS=-D__STDC_CONSTANT_MACROS:$CXXFLAGS cmake -D CMAKE_BUILD_TYPE=RELEASE -D CMAKE_INSTALL_PREFIX=<prefix_path> -D INSTALL_C_EXAMPLES=OFF -D INSTALL_PYTHON_EXAMPLES=ON -D OPENCV_EXTRA_MODULES_PATH=<opencv_contrib_dir>/modules -D BUILD_EXAMPLES=ON -D WITH_OPENCL=ON -D BUILD_opencv_python3=ON -DPYTHON3_INCLUDE_DIR=/share/apps/anaconda3/include/python3.5m  -D PYTHON3_NUMPY_INCLUDE_DIRS=/share/apps/anaconda3/lib/python3.5/site-packages/numpy/core/include -D PYTHON3_EXECUTABLE=/share/apps/anaconda3/bin/python3.5 -D PYTHON3_LIBRARY=/share/apps/anaconda3/lib/libpython3.5m.so -D WITH_EIGEN=ON -D WITH_TBB=ON -D EIGEN_INCLUDE_PATH=/opt/eigen/include/ -DGLOG_INCLUDE_DIRS=<glog_lib_path>/include/ -DGFLAGS_INCLUDE_DIRS=<gflags_lib_path>/include/ -DGLOG_LIBRARIES=<glog_lib_path>/lib/ -DGFLAGS_LIBRARIES=<gflags_lib_path>/lib/ -DBUILD_opencv_dnn=OFF BUILD_opencv_python2=ON -D PYTHON_INCLUDE_DIR=/share/apps/anaconda2/include/python2.7/ -D PYTHON_LIBRARY=/share/apps/anaconda2/lib/libpython2.7.so -D PYTHON2_LIBRARIES=/share/apps/anaconda2/lib/python2.7/ -D PYTHON2_NUMPY_INCLUDE_DIRS=/share/apps/anaconda2/lib/python2.7/site-packages/numpy/core/include -D PYTHON_EXECUTABLE=/share/apps/anaconda2/bin/python2.7 -DWITH_FFMPEG=ON -DBUILD_SHARED_LIBS=ON -DWITH_IPP=OFF -DPYTHON_DEFAULT_EXECUTABLE=/share/apps/anaconda3/bin/python3.5 ..
    $ make -j12
    $ make install
 
So, why do we need to set all these variables in cmake? We have to specify manually where are the python interpreter, default executable, numpy includes, library and include dirs (for python 2.7 and 3), where the opencv\_contrib modules are, where glog, gflags and eigen libraries are. The flag added to CXXFLAG is to avoid undefined macros when compiling a small ffmpeg example.

#### Adding support for FFmpeg

Assuming that FFmpeg was compiled previously and the build directory is <ffmpeg_build>, the following environment variables must be set:

    $ export LD_LIBRARY_PATH=<ffmpeg_build>/lib/:$LD_LIBRARY_PATH
    $ export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:<ffmpeg_build>/lib/pkgconfig  
    $ export PKG_CONFIG_LIBDIR=$PKG_CONFIG_LIBDIR:<ffmpeg_build>/lib/ 
    $ export PATH=<ffmpeg_bin dir>:$PATH

Now, cmake should be able to find FFmpeg through pkg-config tool.

NOTE: FFmpeg must be compiled with the following options: --enable-nonfree --enable-pic --enable-shared

#### Final Steps to correctly setup Python/Anaconda

Now we've compiled both Python 2.7 and 3.5 OpenCV modules. If you are only interested in using one of the Python versions just export the OpenCV python module path to the PYTHONPATH variable:

Add this line to ~/.bashrc to set this automatically for every shell session

    $ export PYTHONPATH=$HOME/<opencv_build>/lib/python2.7(3.5)/site-packages:$PYTHONPATH

To be able to use OpenCV in both Python versions without conflicts we cannot use the PYTHONPATH solution. The Python OpenCV modules are located in:

    $ HOME/<opencv_build>/lib/python2.7/site-packages
    $ HOME/<opencv_build>/lib/python3.5/site-packages

From this point I'll leave to you the decision for which solution to take.

Solution 1: Add the path to python sys.path

    $ python  
    $ >>> import sys  
    $ >>> sys.path.append('$HOME/<opencv_build>/lib/python2.7(3.5)/site-packages')

Solution 2 (more elegant): Add additional site-packages folders, namely, the ones created during OpenCV build.

    $ mkdir -p $HOME/.local/lib/python2.7/site-packages/
    $ mkdir -p $HOME/.local/lib/python3.5/site-packages/  
    $ echo "$HOME/<opencv_build>/lib/python2.7/site-packages" > $HOME/.local/lib/python2.7/site-packages/usrlocal.pth
    $ echo "$HOME/<opencv_build>/lib/python3.5/site-packages" > $HOME/.local/lib/python3.5/site-packages/usrlocal.pth

Then make sure that user site packages are enabled:

    $ unset PYTHONNOUSERSITE

We've created two files usrlocal.pth each containing the path of the respective OpenCV module python version. By default, now Python will now inspect this folders when searching for modules. To confirm this:

    $ python
    $ >>> import sys  
    $ >>> sys.path

Open CV Python directory should be listed in the last command.

To check if it installed correctly:

    $ python  
    >>> import cv2

If the import succeeds then Python-OpenCV is installed.

### Common Problems

#### OpenCV Contrib - Modules Download Failure and Hash mismatch

Read the directory structure described in [9](http://answers.opencv.org/answers/113990/revisions/). Create this structure and download the required files (links are available in the website).

#### IPPICV hash mismatch

While creating the makefile for compilation, the lib ippicv will be automatically downloaded. However, the md5sum of the downloaded file will not match the hardcoded hash on the cmake.

Instead of changing cmake we can manually download the file. Download [URL](https://raw.githubusercontent.com/Itseez/opencv_3rdparty/81a676001ca8075ada498583e4166079e5744668/ippicv/ippicv_linux_20151201.tgz)
Steps:

    $ mkdir <opencv_source>/3rdparty/ippicv/downloads/linux-808b791a6eac9ed78d32a7666804320e && cd "$_"
    $ wget https://raw.githubusercontent.com/Itseez/opencv_3rdparty/81a676001ca8075ada498583e4166079e5744668/ippicv/ippicv_linux_20151201.tgz
    $ cd ../../ && mkdir unpack && cd "$_"
    $ cp ../downloads/linux-808b791a6eac9ed78d32a7666804320e/ippicv_linux_20151201.tgz .
    $ tar zxvf ippicv_linux_20151201.tgz

Alternatively, one can pass the option -D WITH_IPP=OFF to the cmake call to compile without the IPPICV lib.


Caffe
------------

Steps for installing the Caffe [URL](http://caffe.berkeleyvision.org/installation.html). Caffe is a deep learning framework made with expression, speed, and modularity in mind.

Caffe has the following dependencies:

-   Cuda Toolkit and cuDNN (For GPU mode)
-   OpenCV (optional but recommended)
-   BLAS ( ATLAS, MKL, or OpenBLAS)
-   Boost \>= 1.55
-   protobuf, glog, gflags, hdf5

Recently (checked on 27-09-16) caffe compilation files structure changed. These notes have been updated to cope with the new compilation files. Furthermore, instructions for compiling with cmake (which should be easier) were added.

Load necessary modules:

    $ module load cmake gnutools mkl python eigen hdf5 boost mvapich2_eth

First of all, in order to make sure that Make/CMake use the correct compiler set the following environment variables on your current shell session:

    $ export CC=/opt/gnu/gcc/bin/gcc  
    $ export CXX=/opt/gnu/gcc/bin/g++

### Installing missing dependencies

#### glog

Glog is available on git:

    $ git clone git@github.com:google/glog.git  
    $ cd glog

The automake version on the cluster is different from the one that glog is expecting. To fix this:

    $ rm test-driver  
    $ ln -s /opt/gnu/share/automake-1.15/test-driver test-driver

The configure has the aclocal tool version hardcoded aswell (version 1.14). In the rocks cluster the version 1.15 is available. To fix this open the configure file and change the line:

    am__api_version='1.14'

to

    am__api_version='1.15'

Compiling glog:

    $ mkdir build && cd "$_"  
    $ export CXXFLAGS="-fPIC" &&  cmake -DCMAKE_INSTALL_PREFIX=<install-folder> ..  
    $ make VERBOSE=1  
    $ make  
    $ make install

#### gflags

Gflags is available on git:

    $ git clone git@github.com:gflags/gflags.git  
    $ cd gflags

Compiling gflags:

    $ mkdir build && cd "$_"  
    $ export CXXFLAGS="-fPIC" && cmake -DCMAKE_INSTALL_PREFIX=<install-folder> ..  
    $ make  
    $ make install

Make sure that the install-folder is on your PATH and LD\_LIBRARY\_PATH variables.

#### leveldb

Leveldb is available on git:

    $ git clone git@github.com:google/leveldb.git  
    $ cd leveldb  
    $ make  
    $ cp --preserve=links libleveldb.* <install-folder>/lib  
    $ cp -r include/leveldb <install-folder>/include/  
    $ cp --preserve=links out-shared/libleveldb.so* ~/installed_libs/lib/

#### lmdb

Lmdb is available on git and through pip:

    $ pip install lmdb --user  
    $ git clone https://github.com/LMDB/lmdb
    $ cd lmdb/libraries/liblmdb/

Open the Makefile and find the line prefix = /usr/local. Change path to your install folder.

    $ make  
    $ make install

#### protobuf

Lmdb is available on git and through pip:

    $ git clone https://github.com/google/protobuf.git
    $ cd protobuf

Compiling:

    $ ./configure --prefix=<install_folder>  
    $ make  
    $ make check  
    $ make install

### Compiling

Caffe is available on git:

    $ git clone git@github.com:BVLC/caffe.git  
    $ cd caffe  
    $ cp Makefile.config.example Makefile.config

All the options for compiling Caffe are available in the file Makefile.config. We can make the following changes:

*   Use cuDNN on the machine that has an NVIDIA card: Uncomment USE\_CUDNN line.
*   If we have OpenCV version \>= 3: Uncomment OPENCV\_VERSION := 3.
*   Change CUDA dir to /opt/cuda (In a machine with NVIDIA card)
*   Use MKL: set BLAS := mkl
*   BLAS\_INCLUDE := /opt/intel/composer\_xe\_2013\_sp1.2.144/mkl/include
*   BLAS\_LIB := /opt/intel/composer\_xe\_2013\_sp1.2.144/mkl/lib/intel64
*   PYTHON\_INCLUDE := /opt/python/include/python2.7;/opt/python/lib/python2.7/site-packages/numpy/core/include
*   PYTHON\_LIB := /opt/python/lib/
*   Add include and lib gflags and glogs folders to INCLUDE\_DIRS and LIBRARY\_DIRS.
*   Add include and lib opencv folders to INCLUDE\_DIRS and LIBRARY\_DIRS.
*   Add hdf5 path:
    -   Add /opt/hdf5/gnu/mvapich2\_eth/include to INCLUDE\_DIRS
    -   Add /opt/hdf5/gnu/mvapich2\_eth/lib to LIBRARY\_DIRS
*   Add boost libraries path:
    -   Add /opt/boost/gnu/mvapich2\_eth/include to INCLUDE\_DIRS
    -   Add /opt/boost/gnu/mvapich2\_eth/lib to LIBRARY\_DIRS
*   Add /usr/lib64 to LIBRARY\_DIRS
*   Add MPI path:
    -   Add /opt/mvapich2/gnu/eth/include to INCLUDE\_DIRS
    -   Add /opt/mvapich2/gnu/eth/bin to LIBRARY\_DIRS

The result should something like:

    INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /home/dsemedo/installed_libs/include /home/dsemedo/opencv_build/include /opt/hdf5/gnu/mvapich2_eth/include /opt/boost/gnu/mvapich2_eth/include  
    LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /home/dsemedo/installed_libs/lib /home/dsemedo/opencv_build/lib /opt/hdf5/gnu/mvapich2_eth/lib /opt/boost/gnu/mvapich2_eth/lib /usr/lib64

Compile steps:

    $ make  
    $ make install  
    $ make test  
    $ make runtest

Compiling python wrappers:

    $ make pycaffe

Add <caffe_folder>/python to the PYTHONPATH environment variable. To verify that it installed correctly:

    $ python  
    >>> import caffe

### Compiling with CMake

With cmake it is easier to specify which libraries (i.e. which library files) we want to use for compilation. This is crucial to avoid problems related to linking with different library versions.

Steps:

    $ mkdir <caffe_folder>/build  
    $ cmake -D CMAKE_INSTALL_PREFIX=<install_folder> -D CMAKE_PREFIX_PATH=/home/dsemedo/opencv_build -D GFLAGS_LIBRARY=/home/dsemedo/installed_libs/lib/libgflags.a -D GFLAGS_INCLUDE_DIR=/home/dsemedo/installed_libs/include -D GLOG_LIBRARY=/home/dsemedo/installed_libs/lib/libglog.a -D GLOG_INCLUDE_DIR=/home/dsemedo/installed_libs/include -D LMDB_LIBRARIES=/home/dsemedo/installed_libs/lib -D LMDB_INCLUDE_DIR=/home/dsemedo/installed_libs/include -D LevelDB_LIBRARY=/home/dsemedo/installed_libs/lib/libleveldb.so -D LevelDB_INCLUDE=/home/dsemedo/installed_libs/include -D Snappy_LIBRARIES=/home/dsemedo/installed_libs/lib/libsnappy.so -D Snappy_INCLUDE_DIR=/home/dsemedo/installed_libs/include -D BLAS=mkl -DPYTHON_LIBRARY=/opt/python/lib/libpython2.7.so -D NUMPY_INCLUDE_DIR=/opt/python/lib/python2.7/site-packages/numpy/core/include -D LMDB_LIBRARIES=/home/dsemedo/installed_libs/lib/liblmdb.so -D LMDB_INCLUDE_DIR=/home/dsemedo/installed_libs/include -D PROTOBUF_LIBRARY=/home/dsemedo/installed_libs/lib/libprotobuf.so -D PROTOBUF_PROTOC_EXECUTABLE=/home/dsemedo/installed_libs/bin/protoc  ..  
    $ make all -j 12  
    $ make install  
    $ make runtest

The second command is huge and looks very complicated, however, it is only specifying manually which library version we want to use. The CMAKE\_INSTALL\_PREFIX sets the path installation folder and should be defined. To adapt this command, just change the libraries location and include dirs paths and set the installation path.

As with make, in order to get pycaffe working, add <install_folder>/python to the PYTHONPATH environment variable. To verify that it installed correctly:

    $ python  
    >>> import caffe

#### Possible problems

Make was not finding the libsnappy.so so I did the following to solve the problem:

    $ ln -s /usr/lib64/libsnappy.so.1 ~/installed_libs/lib/libsnappy.so

Make failed due to a missing symbol from the libboost\_python.so. Probably the Boost library version available in the cluster was compiled with another python version. I compiled boost into a local folder and used that version.

Boost
------------

The Boost library can be downloaded here: [12](http://www.boost.org/). After downloading, extract it to some folder.

To correctly build the Python boost library using Anaconda some configuration steps are required. Copy the file <boost_folder>/tools/build/example/user-config.jam to your Home folder. Modify the last line as follows:

    using python : 3.5 : /share/apps/anaconda3/bin/python3.5 : /share/apps/anaconda3/include/python3.5m : /share/apps/anaconda3/lib ;

Now boost bootstrap script should identify the correct python. For Python 2.7 modify the paths accordingly.

Compiling boost with support for python 3.5 (Just update the paths and version for python 2.7):

    $ cd <boost_folder>  
    $ ./bootstrap.sh --prefix=<install_folder>   
    $ ./b2 install  --prefix=<install_folder> --enable-unicode=ucs4 -j12

Add the <install_folder>/lib and <install_folder>/include to your PATH.

CUDA
------------

If you ever need a certain version of CUDA to compile a certain package e.g. [NVIDIA Apex](https://nvidia.github.io/apex/). You can install it in your home:

Log in to a compute node so that the GPUs are visile.
Load the gnu and gnutools modules. e.g. module load gnu
Download CUDA package from the [NVIDIA toolkit archive](https://developer.nvidia.com/cuda-toolkit-archive).
You will need to select the correct server architecture. At the time of writing of this tutorial it was:
Linux -> x86_64 -> CentOS -> 6 -> runfile (local)

After selecting the right config you should create the dir for your CUDA installation and use the wget command to download the run file.

For example:    

    $ cd ~ && mkdir -p lib/cuda/10.2 && cd lib/cuda/10.2
    $ wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_rhel6.run
    
Run the downloaded file using the following flags to avoid installing to the default dir (/usr/local/cuda), to which you most likely do not have access.

    $ sh cuda_10.2.89_440.33.01_rhel6.run --toolkit --toolkitpath=$HOME/lib/cuda/10.2
    
The --toolkit flag signals that you are only interested in the toolkit itself, so it will not install drivers, nor samples.
The --toolkit flag specifies the path where the CUDA installation will be in your home.

You should do the same for patches that have been released for that CUDA installation, for example:

    $ wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/patches/1/cuda_10.2.1_linux.run
    $ sh cuda_10.2.1_linux.run --toolkit --toolkitpath=$HOME/lib/cuda/10.2

After the installation is complete you should update the necessary environment variables:

    $ export CUDA_HOME=$HOME/lib/cuda/10.2
    $ PATH=${CUDA_HOME}/bin:${PATH}
    $ LD_LIBRARY_PATH=${CUDA_HOME}/lib64:$LD_LIBRARY_PATH
    
If you want to make this your default CUDA installation you should add it to your $HOME/.bashrc file. Or create a script to easily add the env variables on demand, for a more temporary solution.
