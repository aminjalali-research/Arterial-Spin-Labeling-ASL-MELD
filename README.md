# HCP-ASL Processing Pipeline

This repository provides the HCP-ASL processing pipeline for Arterial Spin Labeling (ASL) data within the HCP framework.

> **Note:** This is a fork of the original pipeline:  
> https://github.com/physimals/hcp-asl


---

## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Installation](#installation)  
   - [Detailed Anaconda/Bash Instructions](./Anaconda_Bash.md)
3. [Software](#software)  
   - [FreeSurfer](#install-freesurfer)  
   - [FSL](#install-fsl)  
   - [Connectome Workbench](#install-connectome-workbench)  
   - [HCP Pipelines](#install-hcp-pipelines)  
   - [Conda Environment & HCP-ASL](#create-conda-environment--install-hcp-asl)  
4. [Verify Installation](#verify-installation)  
5. [Prepare Your Data](#prepare-your-data)  
6. [Run the Pipeline](#run-the-pipeline)  
7. [Running Partial Pipeline Stages](#running-partial-pipeline-stages)

---



## Prerequisites

Before you begin, make sure you have the following software packages installed and their environment variables are set:

- **FreeSurfer (v7.2.0)** ([Installation Guide](https://surfer.nmr.mgh.harvard.edu/fswiki/FS7_linux))
- **FSL (≥ 6.0.5.1)** ([Installation Guide](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslInstallation))
- **Connectome Workbench (≥ v1.5.0)** ([Download](https://humanconnectome.org/software/workbench))
- **HCP Pipelines** (instructions below)
- **Python ≥ 3.9** (via Conda recommended)
- **Others**: `git`, `make`, `gcc`, etc.

---

## Installation 
See the [Detailed Anaconda and Bash environment setup](./Anaconda_Bash.md).

~ (tilde) shows the relative shortcut, pointing out to your home directory (/home/usr). 
Therefore (/home/usr/Music) is equivalent to (~/Music).

To edit use the following methods:
```bash
nano ~/.bashrc
vim ~/.bashrc
gedit ~/.bashrc

After editing, reload the file by:
source ~/.bashrc
```


## Install FreeSurfer
```bash
1. Download and extract the FreeSurfer tarball
$ cd $HOME
$ tar -zxpf freesurfer-linux-centos7_x86_64-7.2.0.tar.gz (Wait till finishes)

2. Move into the extracted directory
$ cd $HOME/freesurfer
$ pwd  # Should return: /home/usr/freesurfer)

3. Set environment variables (type the following command in the terminal)
$ export FREESURFER_DIR=/home/usr/software/freesurfer
$ source $FREESURFER_DIR/SetUpFreeSurfer.sh

4. Verify the installation
$ which freeview (Expected output: /home/usr/freesurfer/bin/freeview)
```

## Install FSL (NeuroImaging)
```bash
# 1. Download the FSL installer
wget https://fsl.fmrib.ox.ac.uk/fsldownloads/fslinstaller.py

# 2. Run the installer
python3 fslinstaller.py

# 3. Add FSL to your shell startup script, enabling command-line usage:
export FSL_DIR=/home/usr/software/fsl/
export PATH=${FSLDIR}/bin:${PATH}
. ${FSLDIR}/etc/fslconf/fsl.sh

export FSLCONFDIR=/home/usr/Software/fsl/config
export FSLDEVDIR=/home/usr/Software/fsl/build

# 4. Verify the installation
which fsl
# Expected output: /usr/local/fsl/bin/fsl
```

## Install Connectome Workbench (NeuroImaging Tool)
```bash
# 1. Download and unpack
tar -zxpf wb_view_linux.tar.gz -C $HOME

# 2. Set environment variables
export CARET7DIR=$HOME/workbench/bin
export PATH=$CARET7DIR:$PATH

# 3. Verify installation
which wb_command
```
## Install HCP Pipelines
```bash
# 1. Clone the HCP pipelines repository
git clone https://github.com/Washington-University/HCPpipelines.git $HOME/HCPpipelines

# 2. Set environment variable
export HCPPIPEDIR=$HOME/HCPpipelines
```
## Create Conda Environment & Install HCP-ASL
Download and install Anaconda through their official website: 
```bash
# 1. Create and activate a new Conda environment
conda create -n hcpasl python=3.11 
conda activate hcpasl

# 2. Install the HCP-ASL package from GitHub
pip install git+https://github.com/physimals/hcp-asl.git
```

## Verify Installation
```bash
# This should print usage information without errors
process_hcp_asl --help
```

## Prepare your Data
Replace SubjectID with your actual subject identifier. Ensure that:

- mbPCASLhr_PA.nii.gz is your multi-band PCASL image.
- PCASLhr_SpinEchoFieldMap_AP.nii.gz and PCASLhr_SpinEchoFieldMap_PA.nii.gz are your spin-echo field maps with anterior-posterior and posterior-anterior phase encoding, respectively.
- coeff_AS82_Prisma.grad is your gradient coefficient file.
```bash
/path/to/study_dir/
├── SubjectID/
    ├── T1w/
    ├── mbPCASLhr_PA.nii.gz
    ├── PCASLhr_SpinEchoFieldMap_AP.nii.gz
    ├── PCASLhr_SpinEchoFieldMap_PA.nii.gz
    └── coeff_AS82_Prisma.grad
```

## Run the Pipeline
With your environment set up and data organized, run the pipeline using the process_hcp_asl command:
```bash
process_hcp_asl \
  --subid 100307 \
  --subdir /path/to/HCP/100307_V1_MR \
  --mbpcasl /path/to/mbPCASLhr_PA.nii.gz \
  --fmap_ap /path/to/PCASLhr_SpinEchoFieldMap_AP.nii.gz \
  --fmap_pa /path/to/PCASLhr_SpinEchoFieldMap_PA.nii.gz \
  --grads /path/to/coeff_AS82_Prisma.grad
Outputs will be placed in:
/path/to/HCP/100307_V1_MR/T1w/ASL
```






























