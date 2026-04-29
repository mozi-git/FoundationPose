FROM nvidia/cuda:11.8.0-devel-ubuntu20.04

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=US/Pacific
ENV CONDA_DIR=/opt/conda
ENV CONDA_ENV=foundationpose
ENV PATH="${CONDA_DIR}/envs/${CONDA_ENV}/bin:${CONDA_DIR}/bin:${PATH}"
ENV TORCH_CUDA_ARCH_LIST="7.5;8.0;8.6;8.9+PTX"
ENV OPENCV_IO_ENABLE_OPENEXR=1
ENV SHELL=/bin/bash

SHELL ["/bin/bash", "--login", "-c"]

RUN ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime && echo ${TZ} > /etc/timezone

RUN apt-get update --fix-missing && \
    apt-get install -y --no-install-recommends \
        bash \
        bzip2 \
        ca-certificates \
        cmake \
        curl \
        g++ \
        gcc \
        git \
        libboost-all-dev \
        libegl1 \
        libeigen3-dev \
        libgl1 \
        libglib2.0-0 \
        libgles2 \
        libglvnd0 \
        libgtk2.0-dev \
        libsm6 \
        libxext6 \
        libxrender1 \
        make \
        ninja-build \
        pkg-config \
        unzip \
        vim \
        wget && \
    rm -rf /var/lib/apt/lists/*

RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /tmp/miniconda.sh && \
    /bin/bash /tmp/miniconda.sh -b -p ${CONDA_DIR} && \
    rm /tmp/miniconda.sh && \
    ln -s ${CONDA_DIR}/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". ${CONDA_DIR}/etc/profile.d/conda.sh" >> ~/.bashrc && \
    (${CONDA_DIR}/bin/conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/main || true) && \
    (${CONDA_DIR}/bin/conda tos accept --override-channels --channel https://repo.anaconda.com/pkgs/r || true) && \
    ${CONDA_DIR}/bin/conda update -n base -c defaults conda -y && \
    ${CONDA_DIR}/bin/conda create -n ${CONDA_ENV} python=3.9 -y && \
    ${CONDA_DIR}/bin/conda clean -afy

WORKDIR /workspace/FoundationPose
COPY . /workspace/FoundationPose

RUN source ${CONDA_DIR}/etc/profile.d/conda.sh && \
    conda activate ${CONDA_ENV} && \
    conda install -y -c conda-forge eigen=3.4.0 && \
    ln -sfn ${CONDA_PREFIX}/include/eigen3 /usr/local/include/eigen3 && \
    python -m pip install --upgrade pip setuptools wheel && \
    python -m pip install --no-cache-dir -r requirements.txt && \
    conda clean -afy

RUN source ${CONDA_DIR}/etc/profile.d/conda.sh && \
    conda activate ${CONDA_ENV} && \
    python -m pip install --quiet --no-cache-dir --no-build-isolation git+https://github.com/NVlabs/nvdiffrast.git && \
    python -m pip install --quiet --no-cache-dir kaolin==0.15.0 -f https://nvidia-kaolin.s3.us-east-2.amazonaws.com/torch-2.0.0_cu118.html && \
    python -m pip install --quiet --no-index --no-cache-dir pytorch3d -f https://dl.fbaipublicfiles.com/pytorch3d/packaging/wheels/py39_cu118_pyt200/download.html

RUN source ${CONDA_DIR}/etc/profile.d/conda.sh && \
    conda activate ${CONDA_ENV} && \
    CMAKE_PREFIX_PATH=${CONDA_PREFIX}/lib/python3.9/site-packages/pybind11/share/cmake/pybind11 bash build_all_conda.sh

RUN echo "conda activate ${CONDA_ENV}" >> ~/.bashrc && \
    ln -sf /bin/bash /bin/sh

CMD ["/bin/bash"]
