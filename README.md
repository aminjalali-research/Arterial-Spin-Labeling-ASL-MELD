# HCP-ASL Processing Pipeline

This repository provides the HCP-ASL processing pipeline for Arterial Spin Labeling (ASL) data within the HCP framework.

Please refer to the original pipeline repository as in: https://github.com/physimals/hcp-asl

---
## Table of Contents

1. [Prerequisites](#prerequisites)  
2. [Install FreeSurfer](#install-freesurfer)  
3. [Install FSL](#install-fsl)  
4. [Install Connectome Workbench](#install-connectome-workbench)  
5. [Install HCP Pipelines](#install-hcp-pipelines)  
6. [Create Conda Environment & Install HCP-ASL](#create-conda-environment--install-hcp-asl)  
7. [Verify Installation](#verify-installation)  
8. [Basic Usage Example](#basic-usage-example)  
9. [Running Partial Pipeline Stages](#running-partial-pipeline-stages)  

---
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

---
