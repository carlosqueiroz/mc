# Usar a imagem oficial do Ubuntu
FROM ubuntu:20.04

# Definir variáveis de ambiente necessárias
ENV DEBIAN_FRONTEND=noninteractive
ENV ROOT_VERSION=v6-32-02
ENV GEANT4_VERSION=v11.2.1
ENV ITK_VERSION=v5.3.0
ENV TORCH_VERSION=1.10.1
ARG GATE_VERSION=v9.4
ENV ROOT_DIR=/root/software/root
ENV GEANT4_DIR=/root/software/geant4
ENV ITK_DIR=/root/software/itk
ENV TORCH_DIR=/root/software/libtorch

# Atualizar pacotes e instalar dependências gerais
RUN apt-get update -qq && \
    apt-get install -y --no-install-recommends \
    gcc \
    g++ \
    make \
    binutils \
    libx11-dev \
    libtbb-dev \
    libxpm-dev \
    libxft-dev \
    libxext-dev \
    qtbase5-dev \
    qt5-qmake \
    git \
    cmake \
    libssl-dev \
    python3.8-dev \
    ccache \
    fftw3-dev \
    wget \
    unzip \
    curl \
    libxml2-dev && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Desabilitar verificação SSL do git (se necessário)
RUN git config --global http.sslVerify false

# Criar diretórios para instalação
WORKDIR /root
RUN mkdir -p $ROOT_DIR $GEANT4_DIR $ITK_DIR $TORCH_DIR /root/software/option_dependencies

# Instalar o ROOT
RUN git clone --branch $ROOT_VERSION https://github.com/root-project/root.git --depth 1 $ROOT_DIR/src && \
    mkdir -p $ROOT_DIR/build && \
    cd $ROOT_DIR/build && \
    cmake $ROOT_DIR/src \
          -DCMAKE_CXX_STANDARD=17 \
          -Dpython=OFF \
          -Dpyroot=OFF \
          -Dclad=OFF \
          -Dxrootd=OFF \
          -DCMAKE_INSTALL_PREFIX=$ROOT_DIR/install && \
    make -j$(nproc) install && \
    rm -rf $ROOT_DIR/src $ROOT_DIR/build

# Instalar o Geant4
RUN git clone --branch $GEANT4_VERSION https://github.com/Geant4/geant4.git --depth 1 $GEANT4_DIR/src && \
    mkdir -p $GEANT4_DIR/build $GEANT4_DIR/data && \
    cd $GEANT4_DIR/build && \
    cmake -DGEANT4_INSTALL_DATA=ON \
          -DGEANT4_BUILD_MULTITHREADED=OFF \
          -DGEANT4_INSTALL_DATADIR=$GEANT4_DIR/data \
          -DCMAKE_INSTALL_PREFIX=$GEANT4_DIR/install \
          $GEANT4_DIR/src && \
    make -j$(nproc) install && \
    rm -rf $GEANT4_DIR/src $GEANT4_DIR/build

# Instalar o ITK com RTK
RUN mkdir -p $ITK_DIR/src $ITK_DIR/build && \
    cd $ITK_DIR/src && \
    git clone --branch $ITK_VERSION https://github.com/InsightSoftwareConsortium/ITK.git --depth 1 . && \
    cd $ITK_DIR/build && \
    cmake -DCMAKE_CXX_FLAGS="-std=c++17" \
          -DBUILD_TESTING=OFF \
          -DITK_USE_FFTWD=ON \
          -DITK_USE_FFTWF=ON \
          -DITK_USE_SYSTEM_FFTW:BOOL=ON \
          -DITK_MINIMUM_COMPLIANCE_LEVEL=3 \
          -DModule_RTK=ON \
          -DREMOTE_GIT_TAG_RTK=master \
          $ITK_DIR/src && \
    make -j$(nproc) && \
    rm -rf $ITK_DIR/src

# Instalar o Torch com certificados atualizados
RUN apt-get update && apt-get install -y ca-certificates && \
    mkdir -p $TORCH_DIR && \
    cd /root/software && \
    wget https://download.pytorch.org/libtorch/cpu/libtorch-cxx11-abi-shared-with-deps-$TORCH_VERSION%2Bcpu.zip && \
    unzip libtorch-cxx11-abi-shared-with-deps-$TORCH_VERSION+cpu.zip -d $TORCH_DIR && \
    rm libtorch-cxx11-abi-shared-with-deps-$TORCH_VERSION+cpu.zip

# Definir variáveis de ambiente para ROOT e Geant4
ENV ROOTSYS=$ROOT_DIR/install
ENV PATH=$ROOTSYS/bin:$PATH
ENV LD_LIBRARY_PATH=$ROOTSYS/lib:$LD_LIBRARY_PATH

# Atualizar CMAKE_PREFIX_PATH para incluir o diretório de instalação do Geant4
ENV CMAKE_PREFIX_PATH=$GEANT4_DIR/install:$CMAKE_PREFIX_PATH

# Fonte dos scripts de ambiente do ROOT e Geant4
RUN echo "source $ROOTSYS/bin/thisroot.sh" >> /root/.bashrc
RUN echo "source $GEANT4_DIR/install/bin/geant4.sh" >> /root/.bashrc

# Clonar o repositório do Gate
RUN git clone https://github.com/OpenGATE/Gate.git /root/software/gate/src && \
    cd /root/software/gate/src && \
    git checkout ${GATE_VERSION}

# Construir o Gate
WORKDIR /root/build
RUN cmake -DGATE_USE_TORCH=ON \
          -DTorch_DIR=$TORCH_DIR/libtorch/share/cmake/Torch \
          -DGATE_USE_RTK=ON \
          -DITK_DIR=$ITK_DIR/build \
          -DGATE_COMPILE_GATEDIGIT=ON \
          -DGATE_USE_OPTICAL=ON \
          -DBUILD_TESTING=ON \
          /root/software/gate/src && \
    make -j$(nproc)

# Definir o comando padrão
CMD ["bash"]