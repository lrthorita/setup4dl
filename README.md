# Setting up your laptop with TensorFlow and CARLA Simulator
This script was tested on a laptop with the following configurations:

- Ubuntu 16.04 (Xenial)
- Intel i7 (7th Gen)
- NVIDIA GeForce GTX 1050Ti (4GB)

Installed drivers and packages:
- GPU Drivers
	- NVIDIA 440.33.01
	- CUDA Version: 10.2.89
	- cudnn Version: 7.6.5
	- nvidia-docker2: 2.2.2-1
- Docker 19.03.5
	- nvidia/cuda:10.2-base
	- tensorflow/tensorflow:2.1.0-gpu-py3
	- carlasim/carla:0.9.7

# GPU Setup
This part was based on the [nvidia-reinstall](https://gist.github.com/morgangiraud/990cf65dcb27068a4ca6b9db4957acc7) script.

## I. Install NVIDIA driver
 - [ ] 1. Remove anything linked to nvidia
	 ```
	 sudo apt-get remove --purge nvidia*
	 sudo apt-get autoremove
	 ```
 - [ ] 2. Search for your driver
    ```apt search nvidia```
 - [ ] 3. Select one driver (the last one is a decent choice)
	`apt install nvidia-440`
 - [ ] 4. Test the driver
	```
	sudo shutdown -r now
	nvidia-smi
	```
> **Note:** If it doesn't work, sometimes this is due to a secure boot option of your motherboard, disable it and test again*

## II. Install cuda 

- [ ] 1. Get your [deb (local) cuda file](https://developer.nvidia.com/cuda-downloads) and install toolkit.
	 ```
	wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-ubuntu1604.pinsudo mv cuda-ubuntu1604.pin /etc/apt/preferences.d/cuda-repository-pin-600
	wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda-repo-ubuntu1604-10-2-local-10.2.89-440.33.01_1.0-1_amd64.debsudo 
	dpkg -i cuda-repo-ubuntu1604-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
	sudo apt-key add /var/cuda-repo-10-2-local-10.2.89-440.33.01/7fa2af80.pub
	sudo apt-get update
	sudo apt-get -y install cuda
	```
- [ ] 2. In `.bashrc` file, add cuda to your `PATH`.
	 ```
	export PATH=/usr/local/cuda-10.2/bin${PATH:+:${PATH}}
	export LD_LIBRARY_PATH=/usr/local/cuda-10.2/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
	export CUDA_HOME=/usr/local/cuda-10.2
	```
- [ ] 3. Use the toolkit to check your CUDA capable devices
	```
	cuda-install-samples-10.2.sh ~/.
	cd ~/NVIDIA_CUDA-10.2_Samples/1_Utilities/deviceQuery
	make
	```
- [ ] 4. Test cuda.
	```
	cd ~/NVIDIA_CUDA-10.2_Samples/1_Utilities/deviceQuery
	./deviceQuery
	```

## III. Install cudnn
- [ ] 1. Downloads cudnn deb files from the [nvidia website](https://developer.nvidia.com/rdp/cudnn-download).
	- Choose [cuDNN Runtime Library for Ubuntu16.04 (Deb)](https://developer.nvidia.com/compute/machine-learning/cudnn/secure/7.6.5.32/Production/10.2_20191118/Ubuntu16_04-x64/libcudnn7_7.6.5.32-1%2Bcuda10.2_amd64.deb).
- [ ] 2. Install.
	```
	tar -zxvf cudnn-9.0-linux-x64-v5.1.tgz 
	sudo mv cuda/include/* /usr/local/cuda-9.0/include/.
	sudo mv cuda/lib64/* /usr/local/cuda-9.0/lib64/.
	```
- [ ] 3. Reload your shell
	`. ~/.bashrc`

# Docker Setup
This part is based on the [Docker's tutorial](https://docs.docker.com/install/linux/docker-ce/ubuntu/), and works if the `overlay2` storage driver is used.
## I. Set up the repository
- [ ] 1. Uninstall old versions.
	```
	sudo apt-get remove docker docker-engine docker.io containerd runc
	```
	> ***Note:*** It’s OK if  `apt-get`  reports that none of these packages are installed.
The contents of  `/var/lib/docker/`, including images, containers, volumes, and networks, are preserved. The Docker Engine - Community package is now called  `docker-ce`.

- [ ] 2. Update the  `apt`  package index:
    ```
    $ sudo apt-get update
    ```
- [ ] 3. Install packages to allow  `apt`  to use a repository over HTTPS:
    ```
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        gnupg-agent \
        software-properties-common
    ```
- [ ] 4. Add Docker’s official GPG key:
    ```
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```
- [ ] 5. Verify that you now have the key with the fingerprint  `9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`, by searching for the last 8 characters of the fingerprint.
    ```
    $ sudo apt-key fingerprint 0EBFCD88
        
    pub   rsa4096 2017-02-22 [SCEA]
          9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
    uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
    sub   rsa4096 2017-02-22 [S]
    ```
- [ ] 6. Use the following command to set up the  **stable**  repository.
	```
	sudo add-apt-repository \
	   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
	   $(lsb_release -cs) \
	   stable"
	   ```

## II. Install Docker
- [ ] 1. Update the  `apt`  package index.
    ```
    sudo apt-get update
    ```
- [ ] 2.  Install the  _latest version_  of Docker Engine - Community and containerd, or go to the next step to install a specific version:
    ```
    sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```
- [ ] 3.  Verify that Docker Engine - Community is installed correctly by running the  `hello-world`  image.
    ```
    sudo docker run hello-world
    ```
	This command downloads a test image and runs it in a container. When the container runs, it prints an informational message and exits.

## III. Install [NVIDIA Container Toolkit](https://github.com/NVIDIA/nvidia-docker)

- [ ] 1. Add the package repositories:
	```
	distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
	curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
	curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
	```
- [ ] 2. Update repository and install `nvidia-docker2`:
		```
		sudo apt-get update && sudo apt-get install -y nvidia-docker2
		```
- [ ] 3. Restart docker:
		```
		sudo systemctl restart docker
		```

# Install Docker Images
- [ ]  1. NVIDIA image
```docker pull nvidia/cuda:10.2-base```

- [ ]  2. TensorFlow image
```docker pull tensorflow/tensorflow:2.1.0-gpu-py3```

- [ ]  3. CARLA Simulator image
```docker pull carlasim/carla:0.9.7```
