# HCP-ASL Processing Pipeline

This repository provides the HCP-ASL processing pipeline for Arterial Spin Labeling (ASL) data within the HCP framework.

> **Note:** This is a fork of the original pipeline:  
> https://github.com/physimals/hcp-asl


## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Installation](#installation)  
   - [FreeSurfer](#install-freesurfer)  
   - [FSL](#install-fsl)  
   - [Connectome Workbench](#install-connectome-workbench)  
   - [HCP Pipelines](#install-hcp-pipelines)  
   - [Conda Environment & HCP-ASL](#create-conda-environment--install-hcp-asl)  
3. [Verify Installation](#verify-installation)  
4. [Basic Usage](#basic-usage-example)  
5. [Running Partial Pipeline Stages](#running-partial-pipeline-stages)  


## Prerequisites

Before you begin, make sure you have the following installed and environment variables set:

- **FreeSurfer (v7.2.0)**  
  Download and install following the [Linux CentOS 7 instructions](https://surfer.nmr.mgh.harvard.edu/fswiki/FS7_linux).  
- **FSL (≥ 6.0.5.1)**  
  Download and install from the [FSL website](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslInstallation).  
- **Connectome Workbench (≥ v1.5.0)**  
  Download from the [Human Connectome Project](https://humanconnectome.org/software/workbench).  
- **HCP Pipelines**  
  You will clone this below.  
- **Python ≥ 3.9** (we recommend using Conda)  
- **Other**: `git`, `make`, `gcc`, etc.


## Installation 
## Install FreeSurfer
```bash
1. Download and extract the FreeSurfer tarball
$ cd $HOME
$ tar -zxpf freesurfer-linux-centos7_x86_64-7.2.0.tar.gz (Wait till finishes)

2. Move into the extracted directory
$ cd $HOME/freesurfer
$ pwd  # Should return: /home/usr/freesurfer)

3. Set environment variables (type the following command in the terminal)
$ export FREESURFER_HOME=$HOME/freesurfer
$ source $FREESURFER_HOME/SetUpFreeSurfer.sh

4. Verify the installation
$ which freeview (Expected output: /home/usr/freesurfer/bin/freeview)
```

## Install FSL
```bash
# 1. Download the FSL installer
wget https://fsl.fmrib.ox.ac.uk/fsldownloads/fslinstaller.py

# 2. Run the installer
python3 fslinstaller.py

# 3. Add FSL to your shell startup script (e.g., ~/.bashrc or ~/.zshrc)
export FSLDIR=/usr/local/fsl
. ${FSLDIR}/etc/fslconf/fsl.sh

# 4. Verify the installation
which fsl
# Expected output: /usr/local/fsl/bin/fsl
```

## Install Connectome Workbench
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
```bash
# 1. Create and activate a new Conda environment
conda create -n hcpasl python=3.11 -y
conda activate hcpasl

# 2. Install the HCP-ASL package from GitHub
pip install git+https://github.com/physimals/hcp-asl.git
```

## Verify Installation
```bash
# This should print usage information without errors
process_hcp_asl --help
```

## Basic Usage Example
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
## Running Partial Pipeline Stages
To run only stages 0–8 (for example, if you have an external quantifier):
```bash
process_hcp_asl \
  --subid 100307 \
  --subdir /path/to/HCP/100307_V1_MR \
  --grads /path/to/coeff.grad \
  --mbpcasl /path/to/mbPCASL.nii.gz \
  --fmap_ap /path/to/fmap_AP.nii.gz \
  --fmap_pa /path/to/fmap_PA.nii.gz \
  --stages 0 1 2 3 4 5 6 7 8
```
































