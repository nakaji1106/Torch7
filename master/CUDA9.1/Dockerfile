FROM nvidia/cuda-ppc64le:9.1-cudnn7-devel-ubuntu16.04
MAINTAINER Yasuhiro Nakajima <e35294@jp.ibm.com>

# install base
RUN apt-get update && DEBIAN_FRONTEND=noninteractive  apt-get install -y \
    wget curl git apt-transport-https apt-utils vim htop tree zip unzip \
    openssh-server iputils-ping net-tools language-pack-ja-base language-pack-ja fonts-ipafont-mincho build-essential openjdk-8-jdk  sudo&& \
    apt-get clean

# install python3 module
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    python python3 python3-pip python3-numpy python3-scipy python3-matplotlib python3-dev python3-wheel python3-setuptools && \
    apt-get clean

# upgrade module
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    apt-get clean

#setup python3
RUN update-alternatives --install /usr/bin/python python /usr/bin/python2 10
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3 11

# 1. ------------ install package torch required  ------------------ 

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    libzmq3-dev libssl-dev python-zmq luarocks && \
    apt-get clean
   
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y && \
    apt-get clean
 
# 2. ------------ luarocks setting ------------------ 

RUN luarocks install lzmq
RUN luarocks install penlight
RUN luarocks install env
RUN luarocks install luafilesystem   

# 3. ------------ Build and install Torch7  ------------------

ENV TORCH_NVCC_FLAGS="-D__CUDA_NO_HALF_OPERATORS__"

RUN cd / && \
    git clone https://github.com/torch/distro.git /torch --recursive && \
    cd /torch && \
    ./install-deps &&\
    TORCH_LUA_VERSION=LUA51 ./install.sh

ENV LUA_PATH='/root/.luarocks/share/lua/5.1/?.lua;/root/.luarocks/share/lua/5.1/?/init.lua;/torch/install/share/lua/5.1/?.lua;/torch/install/share/lua/5.1/?/init.lua;./?.lua;/torch/install/lib/lua/5.1/?.lua;/torch/install/lib/lua/5.1/?/init.lua'\
    LUA_CPATH='/root/.luarocks/lib/lua/5.1/?.so;/torch/install/lib/lua/5.1/?.so;./?.so;/torch/install/lib/lua/5.1/loadall.so'\
    PATH=/torch/install/bin:$PATH \
    LD_LIBRARY_PATH=/torch/install/lib:$LD_LIBRARY_PATH \ 
    DYLD_LIBRARY_PATH=/torch/install/lib:$DYLD_LIBRARY_PATH \
    LUA_CPATH='/torch/install/lib/?.so;'$LUA_CPATH    

RUN luarocks install image

# 4. ------------ install nccl  ------------------

RUN cd / && \
   git clone https://github.com/NVIDIA/nccl.git && \
   cd /nccl  && \
   make CUDA_HOME=/usr/local/cuda test 
   
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/nccl/build/lib

#     ------------------------ END --------------------------------

# setup ssh
RUN mkdir /var/run/sshd
RUN echo root:$(echo "passw0rd") | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
RUN sed -i '/^AcceptEnv/s/^/#/' /etc/ssh/sshd_config
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd
RUN echo "export PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:$PATH" >> /root/.bashrc && \
    echo "export LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/nccl/build/lib" >> /root/.bashrc && \
    echo "export PYTHONPATH=/mxnet/python" >> /root/.bashrc

# setup timezone and locale
ENV TZ=Asia/Tokyo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN echo "export LANG=ja_JP.UTF-8" >> /root/.bashrc
RUN echo "export LANGUAGE=ja_JP.UTF-8" >> /root/.bashrc
ENV LANGUAGE=ja_JP
ENV LANG=ja_JP.UTF-8

# import python modules
RUN pip3 install IPython==5.3
RUN pip3 install jupyter
RUN mkdir /root/.jupyter
RUN echo "c.NotebookApp.ip = '*'" > ~/.jupyter/jupyter_notebook_config.py


CMD []
