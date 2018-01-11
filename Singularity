####
# Defines a Singularity container with GPU and MPI enabled Pytorch 
####

BootStrap: debootstrap
OSVersion: xenial
MirrorURL: http://us.archive.ubuntu.com/ubuntu/

%environment
  export PATH=${PATH-}:/usr/lib/jvm/java-8-openjdk-amd64/bin/:/usr/local/cuda/bin
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
  export CUDA_HOME=/usr/local/cuda
  export LD_LIBRARY_PATH=${LD_LIBRARY_PATH-}:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64:/usr/lib
  export PATH="/anaconda3/bin:$PATH"

%post
  apt update
  apt-get install -y software-properties-common libc6
  apt-add-repository universe
  apt update
  apt install -y mpich
  apt install -y build-essential wget curl pkg-config libtool autoconf openjdk-8-jdk g++ zip zlib1g-dev unzip git

  # Install Anaconda Python 3
  cd /
  wget https://repo.continuum.io/archive/Anaconda3-5.0.1-Linux-x86_64.sh
  bash Anaconda3-5.0.1-Linux-x86_64.sh -b -p /anaconda3
  PATH="/anaconda3/bin:$PATH"
  rm Anaconda3-5.0.1-Linux-x86_64.sh

  # Install CUDA toolkit and driver libraries/binaries

  # Fetch cuda toolkit installer
  wget http://developer.download.nvidia.com/compute/cuda/7.5/Prod/local_installers/cuda_7.5.18_linux.run

  export PERL5LIB=.
  sh cuda_7.5.18_linux.run --silent --toolkit --override

  # Install cuDNN
  wget http://developer.download.nvidia.com/compute/redist/cudnn/v5.1/cudnn-7.5-linux-x64-v5.1.tgz
  tar xvzf cudnn-7.5-linux-x64-v5.1.tgz
  cp -P cuda/include/cudnn.h /usr/local/cuda/include
  cp -P cuda/lib64/libcudnn* /usr/local/cuda/lib64
  chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

  # Clean up CUDA install
  rm -rf cuda_7.5.18_linux.run
  rm -rf cudnn-7.5-linux-x64-v5.1.tgz

  # Patch CUDA/7.5 to use gcc/4.9, the highest support release
  apt install -y gcc-4.9 g++-4.9
  ln -s /usr/bin/gcc-4.9 /usr/local/cuda/bin/gcc
  ln -s /usr/bin/g++-4.9 /usr/local/cuda/bin/g++

  # Set CUDA related environment variables
  CUDA_HOME=/usr/local/cuda
  LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64

  # Java cert update
  apt install ca-certificates-java
  update-ca-certificates -f

  # Install Additional deeplearning python packages
  
  # Pytorch
  conda install pytorch torchvision cuda75 -c pytorch
  
  # Scikit-Learn
  pip install scikit-learn
  
  # MPI
  pip install mpi4py
  
  # Install Scikit-Optimize
  cd
  git clone https://github.com/scikit-optimize/scikit-optimize.git
  cd scikit-optimize
  pip install .
  cd

  # Install Hyperspace
  cd
  git clone https://github.com/yngtodd/hyperspace.git
  cd hyperspace
  pip install .
  cd

  # Patch container to work on Titan
  cd /
  wget https://raw.githubusercontent.com/olcf/SingularityTools/master/Titan/TitanBootstrap.sh
  sh TitanBootstrap.sh
  rm TitanBootstrap.sh
