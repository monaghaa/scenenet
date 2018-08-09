Bootstrap: docker
From: nvidia/cuda:8.0-devel-ubuntu16.04

%files
#copy files into container
#Expects the NVIDIA optix script in $PWD. copy to /opt in container
     NVIDIA-OptiX-SDK-4.1.1-linux64-22553582.sh /opt

%post
#build everything you need

apt-get update

#Install required system packages
apt-get install -y git wget cmake unzip vim libx11-dev xorg-dev libglu1-mesa-dev freeglut3-dev libglew1.5 libglew1.5-dev libglu1-mesa libglu1-mesa-dev libgl1-mesa-glx libgl1-mesa-dev libomp-dev gsl-bin libgsl-dev libgsl-dbg libjpeg-dev libpng-dev libtiff-dev libx11-dev libavformat-dev libavdevice-dev libavcodec-dev libavutil-dev libswresample-dev libglu-dev libdc1394-22 libdc1394-22-dev libraw1394-dev libblas-dev liblapack-dev libusb-1.0-0-dev libboost-all-dev gdb xvfb

#Get the SceneNet source code
cd /opt
git clone https://bitbucket.org/dysonroboticslab/scenenetrgb-d/src/cd7857322159

##########################################
#install dependencies for Scene Generator
#and then install Scene Generator
##########################################

#Irrlicht (dependency of chrono)
#build both static and shared libraries. I think Chrono needs shared only.
cd /opt
wget http://downloads.sourceforge.net/irrlicht/irrlicht-1.8.2.zip
unzip irrlicht-1.8.2.zip
cd irrlicht-1.8.2/source/Irrlicht 
#make
make sharedlib CXXFLAGS="-fPIC" CFLAGS="-fPIC"
cd /opt
rm -Rf /opt/irrlicht-1.8.2.zip

#Chrono
#Note: finds openmp but does not use mpi
cd /opt
git clone https://github.com/projectchrono/chrono
cd chrono
git checkout b5af3f50c39cf2f40e4b05b3115ebb23a69ec8c9
mkdir ../chrono_build
cd ../chrono_build
cmake -DCH_IRRLICHTDIR=/opt/irrlicht-1.8.2 -DCH_IRRLICHTLIB=/opt/irrlicht-1.8.2/lib/Linux/libIrrlicht.so.1.8.2 -DCMAKE_INSTALL_PREFIX=/usr/local -DENABLE_MODULE_IRRLICHT:BOOL=ON -DENABLE_MODULE_POSTPROCESS:BOOL=ON -DCMAKE_CXX_FLAGS="-fPIC" /opt/chrono
make
make install
cd /opt
rm -Rf /opt/chrono /opt/chrono_build

#Scene Generator
cd /opt/cd7857322159/scene_generator
sed -i 's/\/path\/to\/your\/chrono\/build/\/usr\/local/g' CMakeLists.txt
mkdir build
cd build
cmake ..
make

##########################################
#install dependencies for Camera Trajectory Generator
#and then install Camera Trajectory Generator
##########################################

#TooN
cd /opt
git clone https://github.com/edrosten/TooN
cd TooN/
./configure && make && make install
cd /opt
rm -Rf /opt/TooN

#libuvc 
cd /opt
git clone git://github.com/ktossell/libuvc.git
cd libuvc
mkdir build
cd build
cmake ..
make
make install
cd /opt
rm -Rf /opt/libuvc


#libcvd
cd /opt

#this was the first version I used. 
#required g++-7 due to bug with default g++-5 compiler
#I've commented it out in favor of another libcvd version (below)
#apt-get install -y software-properties-common
#add-apt-repository -y ppa:ubuntu-toolchain-r/test
#apt-get update
#apt-get install -y g++-7
#git clone https://github.com/edrosten/libcvd
#cd libcvd
#CXX=g++-7 ./configure
#make
#make install

#use this version instead..doesn't have g++5 bug:
git clone https://github.com/ankurhanda/libcvd
cd libcvd
./configure
make
make install
cd /opt
rm -Rf /opt/libcvd

#Pangolin
cd /opt
git clone https://github.com/stevenlovegrove/Pangolin.git
cd Pangolin
mkdir build
cd build
cmake ..
cmake --build .
make install
cd /opt
rm -Rf /opt/Pangolin

#ASSIMP (note "IL" format missing; doesn't build assimp_qt_viewer)
cd /opt
git clone https://github.com/assimp/assimp
cd assimp
mkdir build
cd build
cmake CMakeLists.txt -G 'Unix Makefiles' ..
make 
make install
cd /opt
rm -Rf assimp

cd /opt/cd7857322159/camera_trajectory_generator/
mkdir build
cd build
cmake ..
make

##########################################
#install dependencies for Renderer 
#and then install Renderer 
##########################################

#Eigen
git clone https://github.com/eigenteam/eigen-git-mirror
cd eigen-git-mirror/
mkdir build
cd build
cmake ..
make
make install
cd /opt
rm -Rf /opt/eigen-git-mirror

#NVIDIA Optix:
#@ https://developer.nvidia.com/designworks/optix/downloads/4.1.1/linux64/
#need to download before hand and copy into container
cd /opt
mkdir optix
mv NVIDIA-OptiX-SDK-4.1.1-linux64-22553582.sh optix/
cd optix
chmod +x NVIDIA-OptiX-SDK-4.1.1-linux64-22553582.sh
./NVIDIA-OptiX-SDK-4.1.1-linux64-22553582.sh --skip-license --exclude-subdir

#Renderer
cd /opt/cd7857322159/renderer
mkdir build
cd build
cmake -DOptiX_INSTALL_DIR=/opt/optix ..
make


##NOTES: This was the Dockerfile for the orginal docker container
## It may be useful for understanding what packages are already installed
## It should stay commented out

#ARG repository
#FROM ${repository}:8.0-runtime-ubuntu16.04
#LABEL maintainer "NVIDIA CORPORATION <cudatools@nvidia.com>"
#
#RUN apt-get update && apt-get install -y --no-install-recommends \
#        cuda-core-$CUDA_PKG_VERSION \
#        cuda-misc-headers-$CUDA_PKG_VERSION \
#        cuda-command-line-tools-$CUDA_PKG_VERSION \
#        cuda-nvrtc-dev-$CUDA_PKG_VERSION \
#        cuda-nvml-dev-$CUDA_PKG_VERSION \
#        cuda-nvgraph-dev-$CUDA_PKG_VERSION \
#        cuda-cusolver-dev-$CUDA_PKG_VERSION \
#        cuda-cublas-dev-8-0=8.0.61.2-1 \
#        cuda-cufft-dev-$CUDA_PKG_VERSION \
#        cuda-curand-dev-$CUDA_PKG_VERSION \
#        cuda-cusparse-dev-$CUDA_PKG_VERSION \
#        cuda-npp-dev-$CUDA_PKG_VERSION \
#        cuda-cudart-dev-$CUDA_PKG_VERSION \
#        cuda-driver-dev-$CUDA_PKG_VERSION && \
#    rm -rf /var/lib/apt/lists/*
#
#ENV LIBRARY_PATH /usr/local/cuda/lib64/stubs

