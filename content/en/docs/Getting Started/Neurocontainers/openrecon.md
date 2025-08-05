---
title: "Open Recon"
linkTitle: "Open Recon"
description: >
  Neurodesktop containers can be used in Open Recon
---

These instructions were tested on GitHub Codespaces, and we recommend this as a starting point. 

For a local setup you need Docker (https://www.docker.com/), Python3 and you need to install neurodocker and add it to your path:
```
python -m pip install neurodocker
#the path depends on your local setup
export PATH=$PATH:~/.local/lib/python3.12/site-packages/bin
export PATH=$PATH:~/.local/bin
```

## 1) add the installation of the Python MRD server to any recipe in [https://github.com/neurodesk/neurocontainers](https://github.com/neurodesk/neurocontainers/tree/main/recipes)
Make sure to adjust invertcontrast.py to your pipeline needs (or replace/rename other files from the [Python MRD server](https://github.com/kspaceKelvin/python-ismrmrd-server):
```bash
- include: macros/openrecon/neurodocker.yaml
```

here is an example: https://github.com/NeuroDesk/neurocontainers/tree/main/recipes/openreconexample

Then build the recipe:
```
sf-login openreconexample --architecture x86_64
```

## 2) test the tool inside the container on its own first and then test through MRD server 
convert dicom data to mrd test data:
```bash
cd /opt/code/python-ismrmrd-server
python3 dicom2mrd.py -o input_data.h5 PATH_TO_YOUR_DICOM_FILES
```

start server and client and test application:
```bash
python3 /opt/code/python-ismrmrd-server/main.py -v -r -H=0.0.0.0 -p=9002 -s -S=/tmp/share/saved_data &
# wait until you see Serving ... and the press ENTER
python3 /opt/code/python-ismrmrd-server/client.py -G dataset -o openrecon_output.h5 input_data.h5
```

## 3) submit the container-recipe to the https://github.com/NeuroDesk/neurocontainers/ repository
here is an example: https://github.com/NeuroDesk/neurocontainers/tree/main/recipes/openreconexample

Then the container gets build automatically.

## 4) submit the container to the https://github.com/NeuroDesk/openrecon/ repository
here is an example: https://github.com/NeuroDesk/openrecon/tree/main/recipes/openreconexample



# More detailed instructions for building all of this on Github 
contributed by Kerrin Pine

## Prerequisites
You have a GitHub account (public github.com required, so that a container can be submitted to the public NeuroDesk OpenRecon repository and built)

## Process
1.	Fork neurodesk/neurocontainers to your personal GitHub account (Go to https://github.com/neurodesk/neurocontainers, in the upper right click “Fork”, if prompted, fork to your personal GitHub account.
2.	After forking, go to your forked repo, e.g.: https://github.com/YOUR_USERNAME/neurocontainers
3.	Create a new codespace (In your forked repo, click the green <> Code button, Select “Create codespace on master”)
4.	In the terminal created, run neurodocker --version and you should see something like 2.0.0, good!
5.	Still in the terminal, cd recipes, mkdir projectname (replace with your project name), copy files from recipes/openreconexample to this new directory to give us something to start from
6.	In build.yaml and test.yaml, change all occurences of openreconexample to your own project name and change the openreconexample.py to projectname.py
7.	Follow the instructions in build.yaml to build
8.	Building drops you into the container itself. Follow the instructions in test.yaml to import your own test DICOM data (can be dragged from another window into the folder in Codespaces) into a h5 file for testing.
9.	Continue following the instructions in test.yaml to start the server and send demo data to it (e.g. python3 /opt/code/python-ismrmrd-server/client.py -G dataset -o /buildhostdirectory/output.h5 /buildhostdirectory/b0map.h5 c cbsopenreconexample). You should see an appropriate number of images being sent from the client to the server and an appropriate number returned. The output in e.g. output.h5 can be viewed with the built in H5web.
10.	Clicking control-shift-X or Extensions on the left and installing "niivue" gives you a NII image viewer in Codespaces if you need to check intermediate outputs for troubleshooting.
11.	Once the container has been thoroughly tested and you are happy with it, your new files will need to be committed (and pushed if you were not working on github.com), don't include your demo data!
12.	To build a container ready for the scanner, the first step is to open a pull request (e.g. "Add new openreconexample container for MRD server, This PR adds a new example container for OpenRecon using the MRD server. It includes: neurodocker.yaml for building the container, Customized MRD Python scripts, Testing inside GitHub Codespaces".
13.	The second step is to write a recipe for https://github.com/neurodesk/openrecon, again since it's not our repository, we need to fork it, navigate to recipes, create a folder for our project and add an OpenReconLabel.json (defines how the container description and UI options appear on the scanner), and a params.sh with the version number. Then, open a pull request. Updating the version number will trigger the container to be re-built, instructions for downloading and installing the container then appear as an "Issue" on this repository.
