---
title: "Examples"
linkTitle: "Examples"
weight: 1
aliases: 
- /docs/getting-started/local/examples
description: >
  Installation Examples
---

On this page we show specific examples of the different ways of how Neurodesk can be installed on a local computer. We start from the highest level using the Neurodesk app, then go lower level via docker, neurocommand and down to the lowest level using neurocontainers. We also show on each level how containers can be streamed via CVMFS or downloaded locally.

# Running Neurodesk on a Ubuntu 24.04 computer

On a linux machine you have mutliple options to use Neurodesk:
1) Highest abstraction level, and easiest option: Neurodeskapp - this provides you a full Linux desktop with everything configured you need. You do not need to think about Docker or Singularity containers and you can just get your work done. This is the recommended option.
2) High abstraction level: Running Neurodesktop via Docker manually - you still get the desktop with everything configured, but you now have to manage the docker container yourself. This is useful when the app doesn't work well - for example in a remote SSH setup.
3) Middle abstraction level: Use the containers through wrapper scripts on the terminal through Neurocommand - this is great if you don't need a full desktop environment and you want to use the neurodesk tools in your scripts. Neurocommand handles multiple containers for you and you just run your tools as you are used to without having to think about the fact that they are running in singularity/apptainer containers
4) Low abstraction level: Use the containers on the terminal directly - if you just want ot use the containers directly and you want to do everything yourself, that's the best option for you :)

## Highest abstraction level, and easiest option: Neurodeskapp

Download Neurodeskapp: https://github.com/NeuroDesk/neurodesk-app/releases/latest/download/NeurodeskApp-Setup-Debian-x64.deb

and Install:
```bash
sudo apt install ./NeurodeskApp-Setup-Debian-x64.deb
```

Install Docker:
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

sudo chown root:docker /var/run/docker.sock
sudo chmod 666 /var/run/docker.sock
```

After installation run the following command to verify that Docker is working correctly:

```bash
docker run hello-world
```

The Neurodeskapp can be launched directly from the application menu, or by running the `neurodeskapp` command in the command line.

In the Neurodeskapp settings you can choose if you want to stream or download containers to your system.


more information can be found here: https://neurodesk.org/docs/getting-started/local/neurodeskapp/






## High abstraction level: Running Neurodesktop via Docker manually
If you run Ubuntu > 23.10 and you haven't installed the Neurodeskapp before you need to create this apparmor profile under /etc/apparmor.d/neurodeskapp
```bash
echo -e "# This profile allows everything and only exists to give the\n# application a name instead of having the label \"unconfined\"\n\nabi <abi/4.0>,\ninclude <tunables/global>\n\nprofile neurodeskapp \"/opt/NeurodeskApp/neurodeskapp\" flags=(unconfined) {\n  userns,\n\n  # Site-specific additions and overrides. See local/README for details.\n  include if exists <local/neurodeskapp>\n}" | sudo tee /etc/apparmor.d/neurodeskapp
```

you also need to create the ~/neurodesktop-storage folder if you haven't used the app before:
```bash
mkdir -p ~/neurodesktop-storage
```

Make sure you have Docker installed and configured correctly (see Neurodeskapp for instructions), then run in a terminal:

```bash
docker volume create neurodesk-home &&
sudo docker run \
  --shm-size=1gb -it --security-opt apparmor=neurodeskapp --privileged --user=root --name neurodesktop \
  -v ~/neurodesktop-storage:/neurodesktop-storage \
  --mount source=neurodesk-home,target=/home/jovyan \
  -e NB_UID="$(id -u)" -e NB_GID="$(id -g)" \
  -p 8888:8888 \
  -e NEURODESKTOP_VERSION={{< params/neurodesktop/jupyter_neurodesk_version >}} vnmd/neurodesktop:{{< params/neurodesktop/jupyter_neurodesk_version >}}
```

Then open the jupyter link with the token displayed in your browser. Make sure it starts with 127.0.0.1:8888/lab&token=... 

You can also add a flag to the docker command to activate the offline mode: -e CVMFS_DISABLE=true

when finished, make sure to delete the container - otherwise, you will get an error the next time you run the docker command:
```bash
docker rm neurodesktop
```

If you want to pass your GPU into the desktop, first install this on the host:
```bash
# Manually set the distribution to ubuntu22.04 (works with Ubuntu 24.04) - because it doens't exist yet for 24.04
distribution="ubuntu22.04"

# Add the NVIDIA container toolkit repo using the 22.04 version
curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list | \
  sed 's|^deb |deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit.gpg] |' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list > /dev/null

# Update package lists
sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

Then start the neurodesktop container with the GPU flag:
```bash
sudo docker run \
  --shm-size=1gb -it --privileged --user=root --name neurodesktop \
  -v ~/neurodesktop-storage:/neurodesktop-storage \
  -e NB_UID="$(id -u)" -e NB_GID="$(id -g)" \
  --gpus all \
  -p 8888:8888 \
  -e NEURODESKTOP_VERSION={{< params/neurodesktop/jupyter_neurodesk_version >}} vnmd/neurodesktop:{{< params/neurodesktop/jupyter_neurodesk_version >}}
```

then export the --nv flag
```bash
export neurodesk_singularity_opts="--nv" 
```

more information can be found here: https://neurodesk.org/docs/getting-started/neurodesktop/linux/





## Middle abstraction level: Use the containers through wrapper scripts on the terminal through Neurocommand

For this you do not need Docker, but rather Apptainer or Singularity:

```bash
sudo apt-get install -y software-properties-common
sudo add-apt-repository -y ppa:apptainer/ppa
sudo apt-get update
sudo apt-get install -y apptainer
sudo apt-get install -y apptainer-suid
```

Make sure you have Python configured on your system with pip3:
```bash
sudo apt install python3-pip
```


Then install neurocommand:
```bash
cd ~
git clone https://github.com/NeuroDesk/neurocommand.git 
cd neurocommand 
python3 -m venv ./venv
./venv/bin/pip3 install -r neurodesk/requirements.txt
bash build.sh --cli
export APPTAINER_BINDPATH=`pwd -P`
```

now you can search and install containers:
```bash
# this searches for containers and you can install individual containers by running the install commands displayed 
bash containers.sh itksnap

# this installs all containers matching the pattern itksnap
bash containers.sh --itksnap
```

then link the containers directory to the neurodesktop-storage:
```bash
ln -s $PWD/local/containers/ ~/neurodesktop-storage/ 
```

then you can install lmod:
```bash
sudo apt install lmod
```

and configure lmod:

```bash
# Create the module.sh file
sudo bash -c 'cat > /usr/share/module.sh << "EOL"
# system-wide profile.modules                                          #
# Initialize modules for all sh-derivative shells                      #
#----------------------------------------------------------------------#
trap "" 1 2 3

case "$0" in
  -bash|bash|*/bash) . /usr/share/lmod/8.6.19/init/bash ;;
     -ksh|ksh|*/ksh) . /usr/share/lmod/8.6.19/init/ksh ;;
     -zsh|zsh|*/zsh) . /usr/share/lmod/8.6.19/init/zsh ;;
      -sh|sh|*/sh) . /usr/share/lmod/8.6.19/init/sh ;;
          *) . /usr/share/lmod/8.6.19/init/sh ;;  # default for scripts
esac

trap - 1 2 3
EOL'
```

then add the module setup to your ~/.bashrc:
```bash
cat >> ~/.bashrc << 'EOL'
if [ -f '/usr/share/module.sh' ]; then source /usr/share/module.sh; fi

if [ -d /cvmfs/neurodesk.ardc.edu.au/neurodesk-modules ]; then
  module use /cvmfs/neurodesk.ardc.edu.au/neurodesk-modules/*
else
  export MODULEPATH="~/neurodesktop-storage/containers/modules"              
  module use $MODULEPATH
fi
EOL
```

make sure you have set the APPTAINER_BINDPATH to all directories that you want the containers to access
```
export APPTAINER_BINDPATH=`/data,/scratch`
```
you can also add this to your ~/.bashrc


then restart the terminal you can load and run the software using:
```bash
ml itksnap
itksnap
```

If you need nvidia gpu support activate via exporting this environment variable:
```bash
export neurodesk_singularity_opts='--nv'
```

If you get errors like this:
```
/opt/itksnap-4.0.2/lib/snap-4.0.2/ITK-SNAP: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.38' not found (required by /.singularity.d/libs/libGLX.so.0)
/opt/itksnap-4.0.2/lib/snap-4.0.2/ITK-SNAP: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.38' not found (required by /.singularity.d/libs/libEGL.so.1)
/opt/itksnap-4.0.2/lib/snap-4.0.2/ITK-SNAP: /lib/x86_64-linux-gnu/libc.so.6: version `GLIBC_2.38' not found (required by /.singularity.d/libs/libGLdispatch.so.0)
```
This means that the glibc versions inside the container and outside the container are not compatible. You can either disable the GPU flag --nv or use a newer version of the container.

If you do not want to download the containers you can also stream the containers using CVMFS: https://neurodesk.org/docs/getting-started/neurocontainers/cvmfs/

more information: https://neurodesk.org/docs/getting-started/neurocommand/linux-and-hpc/


## Low abstraction level: Use the containers on the terminal directly 

for this you only need apptainer or singularity installed. See above for installation instructions.

Then you can download a container and run it directly:
```bash
#find out which containers are available:
curl -s https://raw.githubusercontent.com/NeuroDesk/neurocommand/main/cvmfs/log.txt

#select a container and download it:
export container=itksnap_3.8.0_20201208
curl -X GET https://neurocontainers.neurodesk.org/$container.simg -O

singularity shell itksnap_3.8.0_20201208.simg
itksnap
```

if you need nvidia GPU support, add --nv:
```bash
singularity shell --nv itksnap_3.8.0_20201208.simg
itksnap
```
If you want to stream the containers, follow these instructions for setting up CVMFS: https://neurodesk.org/docs/getting-started/neurocontainers/cvmfs/

then you can run:
```
singularity shell /cvmfs/neurodesk.ardc.edu.au/containers/itksnap_3.8.0_20201208/itksnap_3.8.0_20201208.simg
```
and the container will be streamed to you :)

More information about Download (offline) mode: https://neurodesk.org/docs/getting-started/neurocontainers/singularity/
