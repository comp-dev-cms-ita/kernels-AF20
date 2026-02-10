#FROM rootproject/root:6.34.00-ubuntu24.04
FROM ubuntu:24.04

# Utils and ROOT dep (git)
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive TZ=Europe/Zurich apt-get -y -q install \
    git curl sudo wget vim 

ENV CONDA_DIR=/opt/conda

RUN curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh"
RUN bash Miniforge3-Linux-x86_64.sh -b -p /opt/conda

# Put conda in path so we can use conda activate
ENV PATH=$CONDA_DIR/bin:$PATH

RUN conda init bash
RUN echo "conda activate base" >> ~/.bashrc

SHELL ["conda", "run", "-n", "base", "/bin/bash", "-c"]

RUN conda install -c conda-forge -y python=3.11 boost-cpp bokeh==3.1.0 xrootd

RUN apt-get -y install davix

# ROOT required deps in Ubuntu/Debian
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive TZ=Europe/Zurich apt-get -y -q install binutils cmake dpkg-dev g++ gcc libssl-dev libx11-dev libxext-dev libxft-dev libxpm-dev libtbb-dev libvdt-dev libgif-dev

# ROOT optional deps in Ubuntu/Debian
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive TZ=Europe/Zurich apt-get -y -q install \
    libglew-dev libftgl-dev xlibmesa-glu-dev \
    libmysqlclient-dev libfftw3-dev libcfitsio-dev \
    graphviz-dev libavahi-compat-libdnssd-dev libldap2-dev \
    libxml2-dev libkrb5-dev libgsl0-dev \
    libblas-dev protobuf-compiler gfortran \
    libpcre3-dev

### ROOT installation from source ###
#RUN git clone https://github.com/root-project/root.git root_src
RUN git clone https://github.com/vepadulano/root.git -b infn-branch root_src
RUN mkdir root_build
RUN cmake -Dgminimal=ON -Dpyroot=ON -Ddataframe=ON -Dgnuinstall=ON -Ddev=ON \
          -Dbuiltin_xrootd=ON -Dxrootd=ON -Dssl=ON -Dbuiltin_davix=ON -Ddavix=ON  \
          -Dtmva=ON -Dmlp=ON -Dtmva-pymva=ON -Dtmva-cpu=ON -Dtmva-sofie=ON \
          -Dasimage=ON -Druntime_cxxmodules=ON -Droot7=ON -Dimt=ON \
          -DCMAKE_CXX_FLAGS="-Wno-error=sign-compare" \
          -B root_build -S root_src
RUN cmake --build root_build -- install -j$(nproc)

# Remove artifacts
RUN rm -rf root_build root_src

# Change environment so that libraries are properly found
RUN echo /usr/local/lib/root >> /etc/ld.so.conf && ldconfig
ENV PYTHONPATH /usr/local/lib/root:$PYTHONPATH
ENV CLING_STANDARD_PCH none

### To be removed ###
#ENV TINI_VERSION v0.19.0
#ADD https://github.com/krallin/tini/releases/download/${TINI_VERSION}/tini /bin/tini
#RUN chmod +x /bin/tini
#RUN python3 -m pip install -U pip setuptools wheel

#RUN curl -fsSL https://research.cs.wisc.edu/htcondor/repo/keys/HTCondor-Release.gpg.key | apt-key add - \
#    && echo  "deb [arch=amd64] https://research.cs.wisc.edu/htcondor/repo/ubuntu/8.9 focal main\ndeb-src https://research.cs.wisc.edu/htcondor/repo/ubuntu/8.9 focal main" > /etc/apt/sources.list.d/htcondor.list

RUN python3 -m pip install \
    htcondor \
    uproot \
    click \
    ipython \
    "dask[complete]==2024.4.1" \
    dogpile.cache \
    ipykernel \
    ipywidgets \
    matplotlib \
    rucio-clients

WORKDIR /usr/local/share/comp-dev-cms-ita

RUN git clone https://github.com/dask/dask-jobqueue.git
RUN cd /usr/local/share/comp-dev-cms-ita/dask-jobqueue \
    && python3 -m pip install -U .

# Install dask remote jobqueue
#RUN git clone --branch AF20-experimental https://github.com/ttedeschi/dask-remote-jobqueue
#RUN git clone --branch  af20-jobqueue https://github.com/comp-dev-cms-ita/dask-remote-jobqueue
RUN git clone --branch  af20 https://github.com/comp-dev-cms-ita/dask-remote-jobqueue
# WORKDIR /usr/local/share/comp-dev-cms-ita/dask-remote-jobqueue
# RUN python3 setup.py install
# WORKDIR /usr/local/share/comp-dev-cms-ita
RUN cd /usr/local/share/comp-dev-cms-ita/dask-remote-jobqueue \
    && python3 -m pip install -U .

COPY ./condor_config /etc/condor/condor_config
COPY ./ca.crt /ca.crt

#RUN apt-key adv --keyserver hkp://pgp.surfnet.nl --recv-keys ACDFB08FDC962044D87FF00B512839863D487A87

#RUN apt-get -y install software-properties-common && add-apt-repository "deb http://repo.data.kit.edu/ubuntu/bionic ./" && add-apt-repository universe

#RUN apt-get -y install wget 

#RUN wget http://archive.ubuntu.com/ubuntu/pool/universe/q/qrencode/libqrencode3_3.4.4-1build1_amd64.deb && dpkg -i libqrencode3_3.4.4-1build1_amd64.deb

RUN apt-get update && DEBIAN_FRONTEND=noninteractive TZ=Europe/Zurich apt-get -y install oidc-agent-cli

RUN openssl rand -writerand /root/.rnd

ENV X509_CERT_DIR=/cvmfs/cms.cern.ch/grid/etc/grid-security/certificates

#RUN python3 -m pip install correctionlib
#RUN conda install -y  correctionlib
RUN conda install -y -c conda-forge correctionlib

ENV correctionlib_DIR /opt/conda/lib/python3.11/site-packages/correctionlib/cmake/
ENV CPATH /opt/conda/lib/python3.11/site-packages/correctionlib/include

RUN git clone https://gitlab.cern.ch/cp3-cms/CMSJMECalculators.git -b 0.2.0
RUN sed -i 's/std::uint32_t/uint32_t/g' CMSJMECalculators/interface/JMESystematicsCalculators.h
RUN sed -i 's/find_package(Python)/find_package(Python 3.11 REQUIRED COMPONENTS Interpreter Development)/g' CMSJMECalculators/CMakeLists.txt
RUN cmake -DCMAKE_CXX_FLAGS="-std=c++11 -include cstdint"  -DPython3_EXECUTABLE=/opt/conda/bin/python3 -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/opt/conda CMSJMECalculators
RUN make
RUN make install

#RUN python3 -m pip install git+https://gitlab.cern.ch/cp3-cms/CMSJMECalculators.git@0.2.0

RUN wget https://go.dev/dl/go1.23.3.linux-amd64.tar.gz
RUN rm -rf /usr/local/go && tar -C /usr/local -xzf go1.23.3.linux-amd64.tar.gz
ENV PATH=$PATH:/usr/local/go/bin

RUN git clone https://github.com/dmwm/dasgoclient.git
WORKDIR dasgoclient
RUN go get github.com/dmwm/cmsauth
RUN go get github.com/dmwm/das2go
RUN go get github.com/vkuznet/x509proxy
RUN go get github.com/buger/jsonparser
RUN go get github.com/pkg/profile

RUN make build_all

RUN mv dasgoclient_amd64 /usr/local/go/bin/dasgoclient

#RUN python3 -m pip install git+https://gitlab.cern.ch/cp3-cms/CMSJMECalculators.git

ENV LD_LIBRARY_PATH=/opt/conda/lib/:/usr/local/share/comp-dev-cms-ita

ENTRYPOINT ["conda", "run", "--no-capture-output", "-n", "base"]
