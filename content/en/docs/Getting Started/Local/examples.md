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
```
sudo apt install ./NeurodeskApp-Setup-Debian-x64.deb
```

Install Docker:
```
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


## High level: Running Neurodesktop via Docker manually
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

Make sure you have Docker installed (see above), then run in a terminal:

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

Then open the jupyter link in your browser. You can also add a flag to the docker command to activate the offline mode: -e CVMFS_DISABLE=true

more information can be found here: https://neurodesk.org/docs/getting-started/neurodesktop/linux/

## Middle level: Use the containers through wrapper scripts on the terminal through Neurocommand

more information: https://neurodesk.org/docs/getting-started/neurocommand/linux-and-hpc/


## Low level: Use the containers on the terminal directly 

More information about CVMFS (online) mode: https://neurodesk.org/docs/getting-started/neurocontainers/cvmfs/
More information about Download (offline) mode: https://neurodesk.org/docs/getting-started/neurocontainers/singularity/
