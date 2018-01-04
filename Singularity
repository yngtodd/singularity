####
# Defines a Singularity container with GPU and MPI enabled TensorFlow
####

BootStrap: debootstrap
OSVersion: xenial
MirrorURL: http://us.archive.ubuntu.com/ubuntu/

%environment
  export PATH=${PATH-}:/usr/lib/jvm/java-8-openjdk-amd64/bin/:/usr/local/cuda/bin
  export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
  export CUDA_HOME=/usr/local/cuda
  export LD_LIBRARY_PATH=${LD_LIBRARY_PATH-}:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64
  #export PATH=/usr/local/cuda/bin:$PATH
  #export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
  export PATH="/anaconda3/bin:$PATH"

%post
  apt update
  apt-get install -y software-properties-common
  apt-add-repository universe
  apt update
  apt install -y mpich
  apt install -y build-essential wget curl pkg-config libtool autoconf openjdk-8-jdk g++ zip zlib1g-dev unzip git
  apt install -y python3-tk python3-numpy python3-scipy python3-dev python3-pip python3-setuptools

  pip3 install --upgrade pip

  # Install Anaconda Python 3
  #cd /
  #wget https://repo.continuum.io/archive/Anaconda3-5.0.0-Linux-x86.sh
  #bash Anaconda3-5.0.0-Linux-x86.sh -b -p /anaconda3
  #PATH="/anaconda3/bin:$PATH"
  #rm Anaconda3-5.0.0-Linux-x86.sh

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

  # Install Bazel
  echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | tee /etc/apt/sources.list.d/bazel.list
  curl https://bazel.build/bazel-release.pub.gpg | apt-key add -
  apt update -y && apt install -y bazel
  apt upgrade -y bazel

  # Make sure no leftover tensorflow artifacts from previous builds
  rm -rf /tmp/tensorflow_pkg
  rm -rf /root/.cache

  # Set tensorflow configure options
  export TF_NEED_MKL=0
  export CC_OPT_FLAGS="-march=native"
  export TF_NEED_JEMALLOC=1
  export TF_NEED_GCP=0
  export TF_NEED_HDFS=0
  export TF_ENABLE_XLA=0
  export TF_NEED_OPENCL=0
  export TF_NEED_CUDA=1
  export TF_CUDA_CLANG=0
  export GCC_HOST_COMPILER_PATH=/usr/bin/gcc-4.9
  export TF_CUDA_VERSION="7.5"
  export CUDA_TOOLKIT_PATH="/usr/local/cuda"
  export TF_CUDNN_VERSION="5"
  export CUDNN_INSTALL_PATH=$CUDA_TOOLKIT_PATH
  export TF_CUDA_COMPUTE_CAPABILITIES="3.5"
  export TF_NEED_VERBS=0
  export TF_NEED_MPI=1
  export MPI_HOME=/usr

  # Tensorflow has horrible MPI support in ./configure...
  ln -s /usr/include/mpi/mpi.h /usr/include/mpi.h
  ln -s /usr/include/mpi/mpio.h /usr/include/mpio.h
  ln -s /usr/include/mpi/mpicxx.h /usr/include/mpicxx.h

  # Java cert update
  apt install ca-certificates-java
  update-ca-certificates -f

  # Install MPI4PY against mpich(python-mpi4py is built against OpenMPI)
  # GCC/4.8 is too old to acept the compile flags required by mpi4py
  pip3 install mpi4py
  
  # Build/Install Tensorflow against python 3
  export PYTHON_BIN_PATH=`which python3`
  export PYTHON_LIB_PATH=/usr/lib/python3/dist-packages
  
  cd /
  git clone https://github.com/tensorflow/tensorflow
  cd tensorflow
  ./configure
  
  bazel --batch build -c opt --config=cuda tensorflow/tools/pip_package:build_pip_package
  bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

  pip3 install /tmp/tensorflow_pkg/tensorflow-*.whl

  #git clone https://github.com/tensorflow/tensorflow.git
  #cd tensorflow
  #git checkout tags/v1.3.0
  #./configure 

  #bazel build -c opt --copt=-mavx --copt=-msse4.1 --copt=-msse4.2 --config=cuda tensorflow/tools/pip_package:build_pip_package --incompatible_disallow_uncalled_set_constructor=false
  #bazel-bin/tensorflow/tools/pip_package/build_pip_package /tmp/tensorflow_pkg

  #pip3 install /tmp/tensorflow_pkg/tensorflow-*.whl

  cd /
  rm -rf tensorflow
  rm -rf /tmp/tensorflow_pkg

  # Install Additional deeplearning python packages
  
  pip3 install keras
  
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
  cd /
  wget https://raw.githubusercontent.com/olcf/SingularityTools/master/Titan/TitanBootstrap.sh
  sh TitanBootstrap.sh
  rm TitanBootstrap.sh

  # Make sure bazel is shutdown so it doesn't stop singularity from cleanly exiting
  bazel shutdown
  #sleep 10
  #pkill -f bazel*
  #ps aux | grep bazel

  echo "All said and done!"
