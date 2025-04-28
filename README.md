# JETSON-AGX-ORIN_ALL_GUIDE:Flashing Jetson AGX ORIN with Virtual Machine and Run different Samples / Pytorch for Jetpack 6.2:
Nvidia Support GUIDE uses Native Ubuntu, But we can use Virtualmachine instaed let's go:

System Requirements: 

-Virtual Machine (VM) Vmware player Workstation 17 (via this link: https://www.techspot.com/downloads/189-vmware-workstation-for-windows.html with  Ubuntu kernel version 22.04

-Don't use VirtualBox ==>> fails to flash ( I had a lot of errors in the Python file Tegra.py )
 
 
**Install SDKMANAGER software :**

-Install SDKManager version 2.2.0, you can follow the steps indicated in the link here(https://docs.nvidia.com/sdk-manager/install-with-sdkm-jetson/index.html)
-We used the latset version of JETPACK (JETPACK 6.2) 

-Flashing the Jetson AGX Orin with command line because it fails with the SDKmanagersoftware;  
    sudo ./flash.sh jetson-agx-orin-devkit internal and the jetson will boot automaticlly
-Configuration post Flash of JETSON AGX ORIN
 Install jetpack 6.2  manually without sdkmanager with command lines: 

    sudo apt update 
    sudo apt upgarde 
    sudo install nvidia-jetpack 

**Docker Container For Jetpack 6.2:**

there are no Docker installed with the JETPACK 6.2, we need to installed it manually: 

   -git clone https://github.com/jetsonhacks/install-docker.git
   -cd install-docker
   -bash ./install_nvidia_docker.sh
   -bash ./configure_nvidia_docker.sh
   
Finally, we get the version 27.7.1, 

and That's all, 



Follow this link from Nvidia catalog, to launch the docker container: 

 1-Allow external applications to connect to the host's X display:
     xhost +
 2-Run the docker container using the docker command:
    sudo docker run -it --rm --net=host --runtime nvidia -e DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix nvcr.io/nvidia/l4t-jetpack:r36.4.0


**RUN some Samples:**  

  **Test VPI samples** (VPI: Vision Progamming Interface: Library for image processing, uses diffrent backend; pva, cuda, cpu, vic)
   (Convolution / Edge detection / image conversion...)
   apt install cmake
   cp -r /opt/nvidia/vpi*/samples /tmp
   cd /tmp/samples/01-convolve_2d/
   mkdir build
   cd build
   cmake ..
   make -j4
  ./vpi_sample_01_convolve_2d cuda /opt/nvidia/vpi2/samples/assets/kodim08.png
  ./vpi_sample_01_convolve_2d cpu /opt/nvidia/vpi2/samples/assets/kodim08.png

 Result: IMAGE WE CAN show it by 
   
  sudo apt-get install eog
  eog img.png 
  
 **Test CUDNN Samples(MnistClassification)**
 First we have to install with the container:
     apt install libfreeimage libfreeimage-dev
     
     cp -r /usr/src/cudnn_samples_v9/ /tmp/
     cd /tmp/cudnn_samples_v9/conv_sample/
     make
    ./conv_sample
Result of classifcation for (ex. 1 3 5)
Test passed 

**Test TensoRT**
First we have to mount the tensorrt root within the container: 
    sudo docker run -it --rm --net=host --runtime nvidia  DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix -v /usr/srs/tensorrt:/workspace/TensorRT  nvcr.io/nvidia/l4t-jetpack:r36.4.0
    cd /workspace/TensorRT/samples/sampleOnnxMNIST
    make
    ./sample_onnx_mnist
 
**Test CUDA Samples( DeviceQuery)** (12.6 VERSION)

 -For Jetpack 6.2 is different than other previous verions because not all samples are tested with jetpack 6.2:
   First we have to install cuda-samples from this github link:https://github.com/NVIDIA/cuda-samples, and then run these command lines: 

     xhost +
    -We mount the cuda-samples in the docker container
     sudo docker run -it --rm --net=host --runtime nvidia  DISPLAY=$DISPLAY -v /tmp/.X11-unix/:/tmp/.X11-unix -v /usr/local/cuda/cuda-samples:/workspace/cuda-samples  nvcr.io/nvidia/l4t-jetpack:r36.4.0

    -Within the docker container we run:
 
     apt-get update && apt-get install -y --no-install-recommends make g++
     cd /workspace/cuda-samples/Samples/1_Utilities/deviceQuery
     mkdir build
     cd build 
     make -j$(nproc) deviceQuery
     ./deviceQuery

   Result=PASS
     
     
**Pytorch for Jetpack 6.2**

sudo wget https://developer.download.nvidia.com/compute/cusparselt/0.7.1/local_installers/cusparselt-local-tegra-repo-ubuntu2204-0.7.1_1.0-1_arm64.deb
sudo dpkg -i cusparselt-local-tegra-repo-ubuntu2204-0.7.1_1.0-1_arm64.deb
sudo cp var/cusparselt-local-tegra-repo-ubuntu2204-0.7.1/cusparselt-*-keyring.gpg /usr/share/keyrings/
sudo apt-get update
sudo apt-get -y install libcusparselt0 libcusparselt-dev
sudo python3 -m pip install --no-cache https://github.com/SnapDragonfly/pytorch/releases/download/v2.5.1%2Bl4t35.6-cp38-cp38-aarch64/torch-2.5.1+l4t36.4-cp310-cp310-linux_aarch64.whl

pip3 show torch 
version: 2.5.1+l4t36.4 

 

