FROM ubuntu:16.04

#Fix default ceres package on 16.04 is broken

RUN apt-get update --quiet && \
    apt-get install --no-install-recommends --no-install-suggests --yes software-properties-common
RUN apt-get update --quiet && \
    apt-get install --no-install-recommends --no-install-suggests --yes  \
    apt-utils wget build-essential python3-dev python3-pip libhdf5-serial-dev cmake git-core \
    libfftw3-dev jq libzmq-dev \
    liblapack-dev libxml2-dev libfreetype6-dev pkg-config \
    libxslt-dev libace-dev gcc-multilib  \
    libgtest-dev python-dev liblapack-dev liblapacke-dev libatlas-base-dev libplplot-dev libdcmtk-dev \
    supervisor net-tools cpio

#Python stuff
RUN pip3 install --upgrade pip
RUN pip3 install -U pip setuptools
RUN apt-get install --no-install-recommends --no-install-suggests --yes python3-psutil python3-pyxb python3-lxml python3-numpy
RUN apt-get install --no-install-recommends --no-install-suggests --yes python3-pil
RUN apt-get install --no-install-recommends --no-install-suggests --yes python3-configargparse
RUN pip3 install Cython scipy tk-tools matplotlib==2.2.3 scikit-image opencv_python pydicom scikit-learn
RUN apt-get install --no-install-recommends --no-install-suggests --yes python3-tk
RUN pip3 uninstall h5py
RUN apt-get install -y python3-h5py

RUN pip3 install --upgrade tensorflow
RUN pip3 install http://download.pytorch.org/whl/cpu/torch-0.4.1-cp35-cp35m-linux_x86_64.whl 
RUN pip3 install torchvision 
RUN pip3 install tensorboardx visdom

# for embedded python plot, we need agg backend
RUN mkdir -p /root/.config/matplotlib && touch /root/.config/matplotlib/matplotlibrc 
RUN echo "backend : agg" >> /root/.config/matplotlib/matplotlibrc

#OpenBLAS with OpenMP
RUN cd /opt && \
    mkdir debsource && \
    cd debsource && \
    apt-get --no-install-recommends --no-install-suggests --yes build-dep libopenblas-base && \
    apt-get install --no-install-recommends --no-install-suggests --yes build-essential fakeroot devscripts && \
    apt-get source libopenblas-base && \
    cd openblas-0.2.18/ && \
    sed -i "s/NO_WARMUP=1/NO_WARMUP=1 OPENMP=1/g" debian/rules && \
    debchange -i "Compiling with OpenMP support" && \
    debuild -us -uc -i -I && \
    debi && \
    update-alternatives --set libblas.so.3 /usr/lib/openblas-base/libblas.so.3 && \
    cd /opt && \
    rm -rf debsource

#Update gcc to something remotely recent
RUN add-apt-repository ppa:ubuntu-toolchain-r/test
RUN apt-get update
RUN apt-get -y install gcc-6 g++-6
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-6 100 --slave /usr/bin/g++ g++ /usr/bin/g++-6

# Compile boost
RUN cd /opt && \
    wget https://dl.bintray.com/boostorg/release/1.67.0/source/boost_1_67_0.tar.gz && \
    tar -xzvf boost_1_67_0.tar.gz && \
    cd boost_1_67_0 && \
    ./bootstrap.sh --with-python=/usr/bin/python3 --prefix=/usr && \
    ./b2 -j$(nproc) address-model=64 stage && \
    ./b2 install && \ 
    cd /opt && rm -rf ./boost_1_67_0

# since cmake has problems to find python3 and boost-python3
RUN ln -s /usr/lib/libboost_python35.so /usr/lib/libboost_python3.so

# Compile armadillo
RUN cd /opt && \
    wget http://sourceforge.net/projects/arma/files/armadillo-8.600.0.tar.xz && \
    tar xvf armadillo-8.600.0.tar.xz && \
    cd armadillo-8.600.0 && \
    mkdir build && cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release ../ && \
    make -j$(nproc) && \
    make install && \ 
    cd /opt && rm -rf ./armadillo-8.600.0

# Clean up downloads
RUN rm -rf /opt/*

#Set more environment variables in preparation for Gadgetron installation
ENV GADGETRON_HOME=/usr/local \
    ISMRMRD_HOME=/usr/local

ENV PATH=$PATH:$GADGETRON_HOME/bin:$ISMRMRD_HOME/bin \
    LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ISMRMRD_HOME/lib:$GADGETRON_HOME/lib

# Clean up packages.
RUN  apt-get clean && \
   rm -rf /var/lib/apt/lists/*

