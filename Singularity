####
# Defines a Singularity container with GPU and MPI enabled Pytorch
# https://www.tensorflow.org/install/install_sources#tested_source_configurations
####

BootStrap: docker
From: ubuntu:zesty

%environment
  export PATH=${PATH-}:/usr/lib/jvm/java-8-openjdk-amd64/bin/:/usr/local/cuda/bin
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
  export CUDA_HOME=/usr/local/cuda
  export LD_LIBRARY_PATH=${LD_LIBRARY_PATH-}:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64

%post
  apt update
  apt-get install -y software-properties-common
  apt-add-repository universe
  apt update
  apt install -y mpich
  apt install -y build-essential wget curl pkg-config libtool autoconf openjdk-8-jdk g++ zip zlib1g-dev unzip git
  apt install -y python3-numpy python3-scipy python3-dev python3-pip python3-setuptools python3-tk

  pip3 install --upgrade pip

  # Install CUDA toolkit and driver libraries/binaries

  # Fetch cuda toolkit installer
  wget http://developer.download.nvidia.com/compute/cuda/7.5/Prod/local_installers/cuda_7.5.18_linux.run

  export PERL5LIB=.
  sh cuda_7.5.18_linux.run --silent --toolkit --override

  # Install cuDNN
  wget http://developer.download.nvidia.com/compute/redist/cudnn/v6.0/cudnn-7.5-linux-x64-v6.0.tgz
  tar xvzf cudnn-7.5-linux-x64-v6.0.tgz
  cp -P cuda/include/cudnn.h /usr/local/cuda/include
  cp -P cuda/lib64/libcudnn* /usr/local/cuda/lib64
  chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

  # Clean up CUDA install
  rm -rf cuda_7.5.18_linux.run
  rm -rf cudnn-7.5-linux-x64-v6.0.tgz

  # Patch CUDA/7.5 to use gcc/4.9, the highest support release
  apt install -y gcc-4.9 g++-4.9
  ln -s /usr/bin/gcc-4.9 /usr/local/cuda/bin/gcc
  ln -s /usr/bin/g++-4.9 /usr/local/cuda/bin/g++

  # Java cert update
  apt install ca-certificates-java
  update-ca-certificates -f

  # Install MPI4PY against mpich(python-mpi4py is built against OpenMPI)
  # GCC/4.8 is too old to acept the compile flags required by mpi4py
  pip3 install mpi4py

  # PyTorch
  pip3 install http://download.pytorch.org/whl/cu75/torch-0.2.0.post4-cp36-cp36m-linux_x86_64.whl
  pip3 install --no-deps torchvision

  pip3 install scikit-learn

  # Install Scikit-Optimize
  cd
  git clone https://github.com/scikit-optimize/scikit-optimize.git
  cd scikit-optimize
  pip3 install .
  cd

  # Install Hyperspace
  cd
  git clone https://github.com/yngtodd/hyperspace.git
  cd hyperspace
  pip3 install .
  cd

  # Patch container to work on Titan
  wget https://raw.githubusercontent.com/olcf/SingularityTools/master/Titan/TitanBootstrap.sh
  sh TitanBootstrap.sh
  rm TitanBootstrap.sh
