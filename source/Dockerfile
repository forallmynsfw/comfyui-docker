
# Defines build arguments for the versions of PyTorch, CUDA, and cuDNN to use
ARG PYTORCH_VERSION=2.9.1
ARG CUDA_VERSION=12.8
ARG CUDNN_VERSION=9

# This image is based on the latest official PyTorch image, because it already contains CUDA, CuDNN, and PyTorch
FROM pytorch/pytorch:${PYTORCH_VERSION}-cuda${CUDA_VERSION}-cudnn${CUDNN_VERSION}-devel

# Defines build arguments for the versions of ComfyUI and ComfyUI Manager to use
ARG COMFYUI_VERSION=0.20.1

# Installs Git, because ComfyUI and the ComfyUI Manager are installed by cloning their respective Git repositories
RUN apt-get update --assume-yes && \
    apt-get install --assume-yes \
        git \
        sudo \
        libgl1-mesa-glx \
        libglib2.0-0 && \
    rm -rf /var/cache/apt/archives /var/lib/apt/lists/*

# Install triton and sageattention
RUN pip install triton
RUN mkdir /opt/sageattention && \
    cd /opt/sageattention && \
    curl -O https://huggingface.co/Kijai/PrecompiledWheels/resolve/main/sageattention-2.2.0-cp312-cp312-linux_x86_64.whl?download=true && \
    pip install wheel && \
    pip install /opt/sageattention/sageattention-2.2.0-cp312-cp312-linux_x86_64.whl
# 40X0 cards
# ARG TORCH_CUDA_ARCH_LIST="sm_89"
# RUN git clone https://github.com/thu-ml/SageAttention.git /opt/SageAttention && \
#    cd /opt/SageAttention && \
#    python setup.py install

# Clones the ComfyUI repository and checks out the latest release
RUN git clone https://github.com/Comfy-Org/ComfyUI.git /opt/comfyui && \
    cd /opt/comfyui && \
    git checkout "v${COMFYUI_VERSION}"

# Installs the required Python packages for both ComfyUI and the ComfyUI Manager
RUN pip install \
    --requirement /opt/comfyui/requirements.txt \
    --requirement /opt/comfyui/manager_requirements.txt

# Sets the working directory to the ComfyUI directory
WORKDIR /opt/comfyui

# Exposes the default port of ComfyUI (this is not actually exposing the port to the host machine, but it is good practice to include it as metadata,
# so that the user knows which port to publish)
EXPOSE 8188

# Adds the startup script to the container; the startup script will create all necessary directories in the models and custom nodes volumes that were
# mounted to the container and symlink the ComfyUI Manager to the correct directory; it will also create a user with the same UID and GID as the user
# that started the container, so that the files created by the container are owned by the user that started the container and not the root user
ADD entrypoint.sh /entrypoint.sh
ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
