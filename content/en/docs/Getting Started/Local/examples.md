---
title: "Examples"
linkTitle: "Examples"
weight: 1
description: >
  Installation Examples
---

On this page we show specific examples of the different ways of how Neurodesk can be installed on a local computer. We start from the highest level using the Neurodesk app, then go lower level via docker, neurocommand and down to the lowest level using neurocontainers. We also show on each level how containers can be streamed via CVMFS or downloaded locally.

# Running Neurodesk on a Ubuntu 24.04 computer

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
# This profile allows everything and only exists to give the
# application a name instead of having the label "unconfined"

abi <abi/4.0>,
include <tunables/global>

profile neurodeskapp "/opt/NeurodeskApp/neurodeskapp" flags=(unconfined) {
  userns,

  # Site-specific additions and overrides. See local/README for details.
  include if exists <local/neurodeskapp>
}
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

If you want to pass your GPU into the desktop, first install this on the host:
```
sudo apt install nvidia-container-toolkit -y
```

Then start the neurodesktop container with the GPU flag:
```
sudo docker run \
  --shm-size=1gb -it --privileged --user=root --name neurodesktop \
  -v ~/neurodesktop-storage:/neurodesktop-storage \
  -e NB_UID="$(id -u)" -e NB_GID="$(id -g)" \
  --gpus all \
  -p 8888:8888 -e NEURODESKTOP_VERSION=2025-06-10 \
  vnmd/neurodesktop:2025-06-10
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

Create a the new file /usr/share/module.sh 
```bash
sudo vi /usr/share/module.sh
```

with the content:
```bash
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
```

then add this to your ~/.bashrc:
```bash
vi ~/.bashrc
```

Add the following lines to your ~/.bashrc
```bash
if [ -f '/usr/share/module.sh' ]; then source /usr/share/module.sh; fi

if [ -d /cvmfs/neurodesk.ardc.edu.au/neurodesk-modules ]; then
        module use /cvmfs/neurodesk.ardc.edu.au/neurodesk-modules/*
else
        export MODULEPATH="~/neurodesktop-storage/containers/modules"              
        module use $MODULEPATH
fi
```


then restart the terminal you can load and run the software using:
```bash
ml itksnap
itksnap
```

If you need nvidia gpu support activate via exporting this environment variable:
```bash
export neurodesk_singularity_opts='--nv'
```

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

More information about CVMFS (online) mode: https://neurodesk.org/docs/getting-started/neurocontainers/cvmfs/

More information about Download (offline) mode: https://neurodesk.org/docs/getting-started/neurocontainers/singularity/
