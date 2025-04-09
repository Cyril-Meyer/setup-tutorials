# Docker for GPU-Accelerated Deep Learning

The following setup has been made on a Aurora R16 with Windows 11 pro (10.0.26100) and and RTX 4090 (driver version 572.83).
I wanted to stay on windows 11 for proprietary driver reason, the computer is not officially linux compatible.

The aim of the setup is to install a permanent Jupyterlab server which use the GPU.

## Long story short

* Install GPU drivers
* Install WSL 2
* Install docker desktop
* `docker run -it --rm --runtime=nvidia tensorflow/tensorflow:2.16.1-gpu bash`
* [optional: setup jupyterlab](#jupyterlab-server)

## Prerequisites

* NVIDIA drivers
* Install WSL 2 [How to install Linux on Windows with WSL](https://learn.microsoft.com/en-us/windows/wsl/install)
  * `wsl --install`
  * `wsl --install Ubuntu-24.04`

## Docker on windows

* Install docker
  * [Docker Desktop Installer.exe](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)
  * During install, verify that the checkbox (use WSL instead of hyper-v is checked)
  * [Install Docker Desktop on Windows](https://docs.docker.com/desktop/setup/install/windows-install/)

## GPU support

Check if GPU is working
* `docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark`

More informations :
* [GPU support in Docker Desktop](https://docs.docker.com/desktop/features/gpu/)

Check if GPU is detected
* `docker run -it --rm --runtime=nvidia tensorflow/tensorflow:latest-gpu bash`
  * `nvidia-smi`
 
![image](https://github.com/user-attachments/assets/b7621bdb-d991-493e-aea4-21570e1625de)

## TensorFlow for deep learning

Run python with tensorflow latest
* `docker run -it --rm --runtime=nvidia tensorflow/tensorflow:latest-gpu python`
  * `import tensorflow as tf`
  * `tf.config.list_physical_devices()`

Personally, when using tensorflow latest gpu it was not working directly (09/04/2025) :
```
W0000 00:00:1744212707.411665      14 gpu_device.cc:2341] Cannot dlopen some GPU libraries. Please make sure the missing libraries mentioned above are installed properly if you would like to use GPU. Follow the guide at https://www.tensorflow.org/install/gpu for how to download and setup the required libraries for your platform.
Skipping registering GPU devices...
[PhysicalDevice(name='/physical_device:CPU:0', device_type='CPU')]
```
I need to add a pip install
* `docker run -it --rm --runtime=nvidia tensorflow/tensorflow:latest-gpu bash`
  * `pip install tensorflow[and-cuda]`
  * `python`
  * `import tensorflow as tf`
  * `tf.config.list_physical_devices()`

```
>>> tf.config.list_physical_devices()
[PhysicalDevice(name='/physical_device:CPU:0', device_type='CPU'), PhysicalDevice(name='/physical_device:GPU:0', device_type='GPU')]
```

Anyway, I currently want to stick with 2.16 for now.

Run python with tensorflow 2.16
* `docker run -it --rm --runtime=nvidia tensorflow/tensorflow:2.16.1-gpu python`
  * `import tensorflow as tf`
  * `tf.config.list_physical_devices()`
 
### Benchmark

Check if GPU work as intended (benchmark)
* `docker run -it --rm --runtime=nvidia tensorflow/tensorflow:2.16.1-gpu bash`
  * `wget https://github.com/Cyril-Meyer/GPUBench-IMAGeS/archive/refs/heads/main.zip`
  * `unzip main.zip`
  * `cd GPUBench-IMAGeS`
  * `python run.py --tensorflow`

My result are the following : 
```
TF2-MLP : 0.5298478841781616
TF2-CNN-ResNet50 : 1.3719675064086914
TF2-CNN-ResNet101 : 2.7556068897247314
```

### Docker Desktop

You can manage the installed images in docker desktop GUI.

![image](https://github.com/user-attachments/assets/ddc3dda9-6ecf-4ade-9e12-d801e8616727)

Now we want a 

More informations
* [Cyril-Meyer/GPUBench-IMAGeS tensorflow benchmark results](https://github.com/Cyril-Meyer/GPUBench-IMAGeS/?tab=readme-ov-file#tensorflow)
* [hub.docker tensorflow/tensorflow](https://hub.docker.com/r/tensorflow/tensorflow/)

## JupyterLab server

Try jupyterlab
* `docker run -it --rm -v .:/tf/notebooks -p 8888:8888 tensorflow/tensorflow:2.16.1-jupyter`

This is the right moment to check if the server is also accessible from another computer (if you want to).

<details>
<summary>Dockerfile</summary>

```dockerfile
FROM tensorflow/tensorflow:2.16.1-gpu

ARG USER_ID
ARG GROUP_ID

RUN groupadd -r -g $GROUP_ID cyril && useradd -r -u $USER_ID -g cyril -m -d /home/cyril cyril
ENV SHELL=/bin/bash
RUN mkdir -p /home/cyril/development && chown -R cyril:cyril /home/cyril/development
WORKDIR /home/cyril/development

RUN apt update
RUN apt install git -y
RUN pip install --upgrade pip
RUN pip install jupyterlab==4.4.0

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--port=8888", "--allow-root"]
```

</details>

* First start in WSL2
  * `docker build --build-arg USER_ID=$(id -u) --build-arg GROUP_ID=$(id -g) -t jlab .`
  * `docker run --runtime=nvidia -it --name jlab-container -v "$(pwd):/home/cyril/development" --user $(id -u):$(id -g) -p 8888:8888 jlab`

After the first start, you can restart in docker desktop:

![image](https://github.com/user-attachments/assets/4e566298-cb5f-46de-ab2f-c72d01059656)

#### Some interesting references

* https://www.pendragonai.com/setup-tensorflow-gpu-windows-docker-wsl2/
* https://www.youtube.com/watch?v=YozfiLI1ogY
