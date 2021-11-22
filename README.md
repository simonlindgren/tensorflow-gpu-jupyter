# Setting up a server for ML in TensorFlow with GPU and Jupyter

If one is neither an engineer nor a developer &ndash; and maybe even then &ndash; setting up a working system for machine learning, using NVIDIA GPU, alongside other packages is tricky. There is huge complexity in terms of cross-compatibilities between the various components, and I got stuck at one point or another with most tutorials I found. For what it is worth, this is a setup that worked for me, where the key turned out to be [Docker](https://www.docker.com/).

### Step 1: Install NVIDIA driver

1. In a computer, with a physical NVIDIA GPU mounted, make a clean install of Ubuntu 20.04 (During the install process: opt to install ssh; opt to install docker).
2. Follow NVIDIA's driver [installation instructions for Ubuntu](https://docs.nvidia.com/datacenter/tesla/tesla-installation-notes/index.html#ubuntu-lts)
3. Reboot and check the driver installation with the `nvidia-smi` command. You should see info about your operational GPU.
4. Follow NVIDIA’s [post-installation-steps](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#post-installation-actions ) (I only did the ‘mandatory’ steps).
----
### Step 2: Test the system with the NVIDIA docker container

Based on [these instructions](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker).

1. Set up `nvidia-docker` by running:

```
distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
       && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
       && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list \
	| sudo tee /etc/apt/sources.list.d/nvidia-docker.list

```


2. Then run 
```
    sudo apt-get update
    sudo apt-get install -y nvidia-docker2
```

3. Restart docker:

```
sudo systemctl restart docker
````


4. Run the `nvidia-smi` command through `nvidia-docker`:

```
sudo docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

You are likely to get a response similar to: `Unable to find image 'nvidia/cuda:11.0-base' locally`. Then just wait (takes some time), for an image to be pulled and installed.

When the test command above gives the same output as a simple `nvidia-smi` (without docker), things are correctly set up.


#### Running Docker images

The stuff above allowed us to run GPU diagnostics (`nvidia-smi`), but we want to perform actual TensorFlow runs on the GPU. The following steps are based on [this tutorial](https://blog.softwaremill.com/setting-up-tensorflow-with-gpu-acceleration-the-quick-way-add80cd5c988).

1. We want to use the [official TensorFlow docker images](https://hub.docker.com/r/tensorflow/tensorflow).
2. We can start a Python prompt within that image by:

```
sudo docker run -it --rm --gpus all tensorflow/tensorflow:latest-gpu python

# Remember to wait for the image to be pulled and installed the first time.
```

3. Under the prompt, check TensorFlow version and GPU:

```
>>> import tensorflow as tf
>>> tf.version.VERSION
>>> tf.config.experimental.list_physical_devices('GPU')
```

4. We want to run TensorFlow with the GPU, and with Jupyter, so we will use the docker image `tensorflow/tensorflow:latest-gpu-jupyter`. We will run that image, which will launch Jupyter on the port we specify (`8888`) . The command below will also assume that a subdirectory named `notebooks` exists in the directory from which the command is run. That subdirectory will be linked into the Docker image.

	For correct permission to the mounted directory, we must run the command below as root, so first `sudo -i`, then:



```
docker run -u $(id -u):$(id -g) -it  --rm --gpus all -p 8888:8888 -v notebooks:/tf/notebooks:z tensorflow/tensorflow:latest-gpu-jupyter
```

-	It will start a Jupyter server
	-	Copy the url which is something like `http://127.0.0.1:8888/?token=60fad0100bb27135d49b35110770e25ed42faf2df13e1427` 
	-	Tunnel into the server with ssh: `ssh -NL 8888:localhost:8888 simon@192.168.1.34` from user terminal.
	-	On user terminal, paste the copied url into the browser.
