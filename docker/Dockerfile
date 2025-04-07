FROM sulfurheron/nvidia-cuda:9.0-cudnn7-devel-ubuntu16.04-2018-06-08

# Hack to not have tzdata cmdline config during build
RUN ln -fs /usr/share/zoneinfo/Europe/Amsterdam /etc/localtime

# === 添加缺失的 NVIDIA GPG 密钥 ===
# 方法1：直接获取指定 Key ID (推荐)
RUN apt-get update && apt-get install -y --no-install-recommends gnupg curl ca-certificates && \
    apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/3bf863cc.pub && \
    # 清理，防止增加镜像大小
    rm -rf /var/lib/apt/lists/*
# 方法2：或者，如果上面的 .pub 文件 URL 不稳定，可以尝试从 keyserver 获取
# RUN apt-get update && apt-get install -y --no-install-recommends gnupg && \
#     apt-key adv --keyserver keyserver.ubuntu.com --recv-keys A4B469963BF863CC && \
#     apt-get purge --auto-remove -y gnupg && \
#     rm -rf /var/lib/apt/lists/*
# ====================================

# Install python3.7 and dependencies, taken from:
# - hhttps://websiteforstudents.com/installing-the-latest-python-3-7-on-ubuntu-16-04-18-04/
# - https://github.com/zju3dv/pvnet/blob/master/docker/Dockerfile
# - https://github.com/zju3dv/clean-pvnet
RUN apt-get update && \
    apt install -yq software-properties-common && \
    add-apt-repository ppa:deadsnakes/ppa && \
    # 注意：这里的 apt update 是必须的，因为它需要获取 deadsnakes PPA 的包列表
    apt-get update && \
    apt install -yq \
        nano \
        sudo \
        wget \
        curl \
        build-essential \
        cmake \
        git \
        ca-certificates \
        python-qt4 \
        libjpeg-dev \
        zip \
        unzip \
        libpng-dev \
        libeigen3-dev \
        libglfw3-dev \
        libglfw3 \
        libgoogle-glog-dev \
        libsuitesparse-dev \
        libatlas-base-dev \
        && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# (mini)conda
# https://repo.anaconda.com/miniconda/
# 注意：使用较新版本的 Miniconda 可能更稳定，但 py37_4.8.3 应该没问题
RUN wget -q https://repo.anaconda.com/miniconda/Miniconda3-py37_4.8.3-Linux-x86_64.sh && \
    sh ./Miniconda3-py37_4.8.3-Linux-x86_64.sh -b -p /opt/conda && \
    rm ./Miniconda3-py37_4.8.3-Linux-x86_64.sh && \
    # 更新 PATH 以便 conda 命令可用
    export PATH="/opt/conda/bin:$PATH" && \
    conda update -n base -c defaults conda && \
    conda install conda-build

# 设置环境变量，使其在后续层和容器运行时生效
# 注意：这里的 PATH 顺序，优先使用 conda 环境的 bin
ENV PATH=/opt/conda/envs/pvnet/bin:/opt/conda/bin:$PATH

# installing PVnet dependencies (and removing pvnet again)
RUN cd /opt && \
    git clone https://github.com/Rachel1477/PVnet_success.git pvnet && \
    cd pvnet && \
    # 使用 bash -c 来执行包含 conda activate 的命令序列
    # 确保 conda 环境在 RUN 指令的子 shell 中被正确激活
    bash -c ". /opt/conda/etc/profile.d/conda.sh && \
    conda create -n pvnet python=3.7 -y && \
    conda activate pvnet && \
    pip install torch==1.1.0 torchvision==0.3.0 -f https://download.pytorch.org/whl/cu90/torch_stable.html && \
    pip install Cython==0.28.2 && \
    pip install numpy \
    pip install -r requirements.txt && \
    pip install transforms3d" && \
    # 清理
    cd .. && \
    rm -rf pvnet

# 默认命令
CMD ["/bin/bash"]