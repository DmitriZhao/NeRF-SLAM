FROM nvcr.io/nvidia/pytorch:22.12-py3
ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai LANG=C.UTF-8 LC_ALL=C.UTF-8 PIP_NO_CACHE_DIR=1 PYTHONUNBUFFERED=1 PYTHONFAULTHANDLER=1 PYTHONHASHSEED=0 TORCH_CUDA_ARCH_LIST="8.6;8.9" MAX_JOBS=32 CMAKE_VERSION=3.21.0
ENV PATH="${PATH}:/opt/hpcx/ompi/bin"
ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/opt/hpcx/ompi/lib"

RUN sed -i "s/archive.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    sed -i "s/security.ubuntu.com/mirrors.ustc.edu.cn/g" /etc/apt/sources.list &&\
    rm -f /etc/apt/sources.list.d/* &&\
    apt-get update && apt-get upgrade -y &&\
    apt-get install -y --no-install-recommends \
        autoconf automake autotools-dev build-essential ca-certificates \
        make cmake ninja-build yasm pkg-config checkinstall ccache doxygen gdb \
        gcc gcc-multilib g++ g++-multilib gfortran \
        git nano curl wget unzip p7zip-full net-tools cloc htop iftop iotop ffmpeg \
        libatlas-base-dev libboost-all-dev libcereal-dev libcgal-dev \
        libcxsparse3 libeigen3-dev liblapack-dev liblapacke-dev \
        metis libmetis-dev libopenblas-dev libsuitesparse-dev \
        libgtk2.0-dev libgtk-3-dev libglew-dev libgl1-mesa-dev libgl1-mesa-glx mesa-utils \
        libgflags-dev libgoogle-glog-dev libgtest-dev libprotobuf-dev protobuf-compiler \
        libgphoto2-dev libhdf5-dev libjpeg-dev libpng-dev libtiff-dev libtiff-dev \
        libavcodec-dev libavformat-dev libavutil-dev libavdevice-dev libavresample-dev \
        libpostproc-dev libpq-dev libswscale-dev ffmpeg \
        libtbb2 libtbb-dev libxine2-dev zlib1g-dev \
        libkrb5-dev libssl-dev libtool libyaml-dev libyaml-cpp-dev \
        libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev \
        libfaac-dev libmp3lame-dev libvorbis-dev \
        libopencore-amrnb-dev libopencore-amrwb-dev \
        libv4l-dev v4l-utils \
        libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev \
        colmap &&\
    apt-get clean

# Install TBB & pip stuff
ARG TBB_BASEURL=https://github.com/oneapi-src/oneTBB/releases/download
ARG TBB_VERSION=4.4.5
ARG TBB_DIR=tbb44_20160526oss
ARG TBB_SAVEPATH="/tmp/tbb.tgz"
ARG TBB_LIB_DIR="intel64/gcc4.4"
ARG TBBROOT=/tmp/$TBB_DIR
COPY requirements.txt /tmp
RUN wget "${TBB_BASEURL}/${TBB_VERSION}/${TBB_DIR}_lin.tgz" -O $TBB_SAVEPATH &&\
    tar -C /tmp -xf "${TBB_SAVEPATH}" &&\
    cp -R $TBBROOT/lib/$TBB_LIB_DIR/* /usr/local/lib/ &&\
    cp -R $TBBROOT/include/ /usr/local/include/ &&\
    pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple &&\
    pip install -U pip &&\
    pip install cmake==${CMAKE_VERSION} &&\
    pip install -r /tmp/requirements.txt &&\
    cd /tmp &&\
    git clone https://github.com/pytorch/vision.git &&\
    cd vision &&\
    git checkout tags/v0.15.1 &&\
    python setup.py install &&\
    cd /tmp &&\
    git clone https://github.com/rusty1s/pytorch_scatter &&\
    cd pytorch_scatter &&\
    git checkout tags/2.1.1 && python setup.py install &&\
    rm -rf /tmp/*

# Build NeRF-SLAM
WORKDIR /workspace
RUN git clone https://github.com/DmitriZhao/NeRF-SLAM --recurse-submodules &&\
    cd NeRF-SLAM &&\
    git submodule update --init --recursive &&\
    cmake ./thirdparty/instant-ngp -B build_ngp &&\
    cmake --build build_ngp --config RelWithDebInfo -j &&\
    cmake ./thirdparty/gtsam -DGTSAM_BUILD_PYTHON=1 -B build_gtsam &&\
    cmake --build build_gtsam --config RelWithDebInfo -j -t install python-install python-test &&\
    python setup.py install
