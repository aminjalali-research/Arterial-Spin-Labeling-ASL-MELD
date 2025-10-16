
# HCP ASL Processing Pipeline

## Overview

The HCP ASL (Human Connectome Project Arterial Spin Labeling) processing pipeline is a toolset for processing ASL MRI data. This pipeline performs motion correction, distortion correction, empirical banding correction, perfusion quantification, and surface projection of ASL data.

## Table of Contents
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Environment Setup](#environment-setup)
- [Basic Usage](#basic-usage)
- [Command Line Interface](#command-line-interface)
- [Pipeline Stages](#pipeline-stages)
- [Input Data Requirements](#input-data-requirements)
- [Output Structure](#output-structure)
- [Common Workflows](#common-workflows)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Software

Before installing the HCP ASL pipeline, ensure you have the following prerequisites installed:

1. **FSL (≥ 6.0.5.1)** - [Installation Guide](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/FslInstallation)
   - Required for `fabber` and `oxford_asl_roi_stats.py` features
   - Set `FSLDIR` environment variable

2. **Connectome Workbench (≥ 1.5.0)** - [Download](https://www.humanconnectome.org/software/get-connectome-workbench)
   - Required for surface projection with `-weighted` option
   - Set `CARET7DIR` environment variable pointing to directory containing `wb_command`

3. **HCP Pipelines** - [GitHub Repository](https://github.com/Washington-University/HCPpipelines)
   - Required for (Sub-)Cortical LUTs in SE-based bias correction
   - Set `HCPPIPEDIR` environment variable

4. **FreeSurfer** - [Installation Guide](https://surfer.nmr.mgh.harvard.edu/fswiki/DownloadAndInstall)
   - Required for boundary-based registration (BBR)
   - Set `FREESURFER_HOME` environment variable

### System Requirements

- **Python**: ≥ 3.9
- **Memory**: Minimum 8GB RAM (16GB+ recommended for large datasets)
- **Storage**: 10-50GB free space per subject depending on processing stages
- **OS**: Linux or macOS (Windows via WSL)

## Installation

### Method 1: Conda Environment (Recommended)

```bash
# Create and activate conda environment
conda create -n hcp-asl python=3.9
conda activate hcp-asl

# Install from PyPI (when available)
pip install hcp-asl

# OR install from source
pip install git+https://github.com/physimals/hcp-asl.git
```

### Method 2: Using the provided environment file

If you have the repository cloned:

```bash
cd /path/to/hcp-asl
conda env create -f environment.yml
conda activate hcp-asl
```

### Method 3: Development Installation

For development or if you need to modify the code:

```bash
git clone https://github.com/physimals/hcp-asl.git
cd hcp-asl
conda create -n hcp-asl python=3.9
conda activate hcp-asl
pip install -e .
```

### Verification

Test your installation:

```bash
# Check if the main command is available
process_hcp_asl --help

# Verify Python package import
python -c "import hcpasl; print('HCP ASL package successfully installed!')"
```

## Environment Setup

### Required Environment Variables

Set these environment variables in your `.bashrc` or `.bash_profile`:

```bash
# FSL
export FSLDIR=/path/to/fsl
export PATH=${FSLDIR}/bin:${PATH}
source ${FSLDIR}/etc/fslconf/fsl.sh

# Connectome Workbench
export CARET7DIR=/path/to/workbench/bin_linux64

# HCP Pipelines
export HCPPIPEDIR=/path/to/HCPpipelines

# FreeSurfer
export FREESURFER_HOME=/path/to/freesurfer
source ${FREESURFER_HOME}/SetUpFreeSurfer.sh
```

### Verify Environment

```bash
# Test FSL
flirt -version

# Test Workbench
${CARET7DIR}/wb_command -version

# Test FreeSurfer
mri_convert --version
```

## Basic Usage

### Minimal Command

```bash
process_hcp_asl \
  --subid SUBJECT_ID \
  --subdir /path/to/study/SUBJECT_ID \
  --mbpcasl /path/to/mbPCASLhr_PA.nii.gz \
  --fmap_ap /path/to/PCASLhr_SpinEchoFieldMap_AP.nii.gz \
  --fmap_pa /path/to/PCASLhr_SpinEchoFieldMap_PA.nii.gz \
  --grads /path/to/coeff_AS82_Prisma.grad
```

### Quick Processing (Stages 0-8 only)

For custom perfusion quantification:

```bash
process_hcp_asl \
  --subid SUBJECT_ID \
  --subdir /path/to/study/SUBJECT_ID \
  --mbpcasl /path/to/mbPCASLhr_PA.nii.gz \
  --fmap_ap /path/to/PCASLhr_SpinEchoFieldMap_AP.nii.gz \
  --fmap_pa /path/to/PCASLhr_SpinEchoFieldMap_PA.nii.gz \
  --grads /path/to/coeff_AS82_Prisma.grad \
  --stages 0 1 2 3 4 5 6 7 8
```

## Command Line Interface

### Main Command: `process_hcp_asl`

#### Required Arguments

- `--subid`: Subject identifier (e.g., "HCD0001234_V1_MR")
- `--subdir`: Subject's data directory path
- `--mbpcasl`: Path to mbPCASL sequence (.nii.gz)
- `--fmap_ap`: Path to AP phase encoding fieldmap (.nii.gz)
- `--fmap_pa`: Path to PA phase encoding fieldmap (.nii.gz)  
- `--grads`: Path to gradient coefficients file (.grad)

#### Optional Arguments

- `--stages`: Space-separated list of stages to run (default: 0-13)
- `--outdir`: Output directory name (default: "hcp_asl")
- `--cores`: Number of CPU cores to use (default: 1)
- `--nobandingcorr`: Disable banding correction
- `--interpolation`: Interpolation order for registrations (default: 3)
- `--use_t1`: Use estimated T1 maps in oxford_asl

#### Structural Data Arguments

- `--t1`: Path to T1-weighted image
- `--t2`: Path to T2-weighted image  
- `--wmparc`: Path to FreeSurfer wmparc.nii.gz
- `--ribbon`: Path to FreeSurfer ribbon.nii.gz

#### Advanced Arguments

- `--reg_name`: Surface registration sphere (e.g., "MSMAll", "MSMSulc")
- `--territories_atlas`: Path to vascular territories atlas
- `--territories_labels`: Path to vascular territories labels

### Additional Commands

#### SE-based Bias Estimation
```bash
get_sebased_bias_asl \
  --subdir /path/to/subject \
  --fmap_ap /path/to/fmap_ap.nii.gz \
  --fmap_pa /path/to/fmap_pa.nii.gz \
  --wmparc /path/to/wmparc.nii.gz \
  --ribbon /path/to/ribbon.nii.gz
```

#### Magnetization Transfer Estimation
```bash
mt_estimation_asl \
  --subid SUBJECT_ID \
  --subdir /path/to/subject \
  --mt_factors /path/to/mt_factors.txt
```

#### Results to MNI Space
```bash
results_to_mni_asl \
  --subid SUBJECT_ID \
  --subdir /path/to/subject
```

## Pipeline Stages

The pipeline consists of 14 stages (0-13) that can be run independently:

### Stage 0: Data Preparation
- Splits mbPCASL sequence into ASL timeseries and M0 calibration images
- Creates necessary directory structure
- **Outputs**: `label_control.nii.gz`, `calib0.nii.gz`, `calib1.nii.gz`

### Stage 1: Distortion Correction Setup
- Derives gradient and susceptibility distortion correction transforms
- Registers fieldmaps to structural data
- **Outputs**: Gradient unwarp coefficients, TOPUP parameters

### Stage 2: M0 Calibration Correction
- Applies distortion corrections to calibration images
- Performs bias field correction using SE-based methods
- **Outputs**: Corrected calibration images

### Stage 3: ASL Timeseries Correction
- Multi-step correction of ASL timeseries:
  - Empirical banding correction
  - Saturation recovery modeling
  - Slice-timing correction
  - Motion correction
  - Final registration
- **Outputs**: Fully corrected ASL timeseries

### Stage 4: Native Space Subtraction
- Tag-control subtraction in native ASL space
- **Outputs**: Subtracted perfusion images

### Stage 5: Native Space Quantification
- Basic perfusion quantification for registration
- **Outputs**: Perfusion maps for BBR

### Stage 6: Registration to Structural Space
- Registers ASL data to T1w space at ASL resolution
- **Outputs**: Transformed images in structural space

### Stage 7: Partial Volume Estimation
- Estimates partial volume effects
- **Outputs**: GM/WM/CSF probability maps

### Stage 8: Structural Space Subtraction
- Tag-control subtraction in structural space
- **Outputs**: Final subtracted images

### Stage 9: Final Quantification
- Full perfusion quantification with oxford_asl
- Includes partial volume correction
- **Outputs**: Calibrated perfusion maps

### Stage 10: ROI Statistics
- Calculates summary statistics within regions of interest
- **Outputs**: CSV files with ROI measurements

### Stage 11: Surface Projection
- Projects results to cortical surfaces
- **Outputs**: CIFTI files for surface visualization

### Stage 12: Output Organization
- Copies key results to standard locations
- Transforms results to MNI space
- **Outputs**: Organized final results

### Stage 13: Quality Control
- Generates QC report and scene files
- **Outputs**: Visual QC materials

## Input Data Requirements

### Required Files

1. **mbPCASL sequence** (`*.nii.gz`)
   - Multi-band pseudo-continuous ASL with background suppression
   - Should contain both label-control pairs and M0 calibration

2. **Fieldmaps** (pair of `*.nii.gz`)
   - AP and PA phase encoding spin-echo fieldmaps
   - Same slice prescription as ASL data
   - Required for distortion correction

3. **Gradient coefficients** (`*.grad`)
   - Scanner-specific gradient nonlinearity coefficients
   - Provided by scanner manufacturer

### Optional but Recommended

4. **T1-weighted structural** (`T1w_acpc_dc_restore.nii.gz`)
   - High-resolution anatomical image
   - Should be skull-stripped and bias-corrected

5. **FreeSurfer outputs**
   - `wmparc.nii.gz`: White matter parcellation
   - `ribbon.nii.gz`: Cortical ribbon mask
   - Required for SE-based bias correction

### Data Organization

Organize data following HCP conventions:
```
study_dir/
└── SUBJECT_ID/
    ├── unprocessed/
    │   └── 3T/
    │       ├── ASL/
    │       │   ├── SUBJECT_ID_3T_pcasl.nii.gz
    │       │   ├── SUBJECT_ID_3T_SpinEchoFieldMap_AP.nii.gz
    │       │   └── SUBJECT_ID_3T_SpinEchoFieldMap_PA.nii.gz
    │       └── T1w_MPR1/
    └── T1w/
        ├── T1w_acpc_dc_restore.nii.gz
        ├── wmparc.nii.gz
        └── ribbon.nii.gz
```

## Output Structure

### Key Output Files

Located in `SUBJECT_ID/T1w/ASL/`:

#### Corrected Data
- `label_control_corrected.nii.gz`: Fully corrected ASL timeseries
- `label_control_corrected_subtracted.nii.gz`: Motion-aware subtracted ASL
- `calib_corrected.nii.gz`: Corrected calibration image

#### Perfusion Maps (Non-PVE corrected)
- `perfusion_calib.nii.gz`: CBF in ml/100g/min
- `arrival.nii.gz`: Arterial transit time in seconds
- `aCBV_calib.nii.gz`: Arterial cerebral blood volume

#### Perfusion Maps (PVE corrected)
- `pvcorr_perfusion_gm_calib_masked.nii.gz`: GM CBF
- `pvcorr_perfusion_wm_calib_masked.nii.gz`: WM CBF
- `pvcorr_arrival_gm_masked.nii.gz`: GM ATT
- `pvcorr_arrival_wm_masked.nii.gz`: WM ATT

#### Surface Projections
Located in `SUBJECT_ID/MNINonLinear/ASL/`:
- `perfusion_calib_Atlas.dscalar.nii`: Surface perfusion (CIFTI)
- `arrival_Atlas.dscalar.nii`: Surface ATT (CIFTI)
- `pvcorr_*_Atlas.dscalar.nii`: PVE corrected surface maps

#### Quality Control
Located in `SUBJECT_ID/T1w/ASL/ASLQC/`:
- `*_hcp_asl_qc.scene`: Workbench scene file
- `*.png`: QC screenshots

### Directory Structure
See the [Extended pipeline outputs](#extended-pipeline-outputs) section in the original README for complete directory tree.

## Common Workflows

### 1. Standard Full Pipeline

Process complete dataset with all corrections and quantification:

```bash
process_hcp_asl \
  --subid HCD0001234_V1_MR \
  --subdir /data/HCP/HCD0001234_V1_MR \
  --mbpcasl /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_pcasl.nii.gz \
  --fmap_ap /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_SpinEchoFieldMap_AP.nii.gz \
  --fmap_pa /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_SpinEchoFieldMap_PA.nii.gz \
  --grads /usr/local/share/gradients/coeff_AS82_Prisma.grad \
  --cores 4
```

### 2. Custom Quantification Workflow

Process through correction stages only for use with external quantification:

```bash
# Step 1: Run pipeline through stage 8
process_hcp_asl \
  --subid HCD0001234_V1_MR \
  --subdir /data/HCP/HCD0001234_V1_MR \
  --mbpcasl /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_pcasl.nii.gz \
  --fmap_ap /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_SpinEchoFieldMap_AP.nii.gz \
  --fmap_pa /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_SpinEchoFieldMap_PA.nii.gz \
  --grads /usr/local/share/gradients/coeff_AS82_Prisma.grad \
  --stages 0 1 2 3 4 5 6 7 8

# Step 2: Use corrected data with your quantification tool
# Input: HCD0001234_V1_MR/T1w/ASL/label_control_corrected.nii.gz
# Input: HCD0001234_V1_MR/T1w/ASL/calib_corrected.nii.gz
```

### 3. Reprocessing Failed Stages

If a particular stage fails, rerun from that stage onwards:

```bash
# If stage 9 (quantification) failed, rerun stages 9-13
process_hcp_asl \
  --subid HCD0001234_V1_MR \
  --subdir /data/HCP/HCD0001234_V1_MR \
  --mbpcasl /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_pcasl.nii.gz \
  --fmap_ap /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_SpinEchoFieldMap_AP.nii.gz \
  --fmap_pa /data/HCP/HCD0001234_V1_MR/unprocessed/3T/ASL/HCD0001234_V1_MR_3T_SpinEchoFieldMap_PA.nii.gz \
  --grads /usr/local/share/gradients/coeff_AS82_Prisma.grad \
  --stages 9 10 11 12 13
```

### 4. Batch Processing

Process multiple subjects:

```bash
#!/bin/bash
STUDY_DIR="/data/HCP"
GRADS="/usr/local/share/gradients/coeff_AS82_Prisma.grad"

for subdir in ${STUDY_DIR}/HCD*_V1_MR; do
    subid=$(basename ${subdir})
    
    process_hcp_asl \
      --subid ${subid} \
      --subdir ${subdir} \
      --mbpcasl ${subdir}/unprocessed/3T/ASL/${subid}_3T_pcasl.nii.gz \
      --fmap_ap ${subdir}/unprocessed/3T/ASL/${subid}_3T_SpinEchoFieldMap_AP.nii.gz \
      --fmap_pa ${subdir}/unprocessed/3T/ASL/${subid}_3T_SpinEchoFieldMap_PA.nii.gz \
      --grads ${GRADS} \
      --cores 4
done
```

## Troubleshooting

### Common Issues

#### 1. Missing Dependencies

**Error**: `FSL not found` or similar
**Solution**: 
- Verify FSL installation: `which fsl`
- Check environment variables: `echo $FSLDIR`
- Source FSL configuration: `source $FSLDIR/etc/fslconf/fsl.sh`

#### 2. Insufficient Memory

**Error**: Process killed or memory allocation errors
**Solutions**:
- Reduce number of cores: `--cores 1`
- Close other applications
- Process stages individually rather than full pipeline

#### 3. File Not Found Errors

**Error**: `No such file or directory`
**Solutions**:
- Verify all input file paths exist and are readable
- Check file permissions: `ls -la /path/to/file`
- Ensure correct file extensions (.nii.gz for NIFTI files)

#### 4. Registration Failures

**Error**: BBR registration fails
**Solutions**:
- Ensure FreeSurfer is properly installed and `FREESURFER_HOME` is set
- Check that structural data exists and is valid
- Verify wmparc.nii.gz and ribbon.nii.gz are present

#### 5. Oxford ASL Errors

**Error**: Perfusion quantification fails  
**Solutions**:
- Check FSL version (≥6.0.5.1 required)
- Verify fabber installation: `which fabber`
- Check calibration image quality

### Log Files

Check pipeline logs for detailed error information:
- Main log: `SUBJECT_ID/T1w/ASL/SUBJECT_ID_hcp_asl.log`
- Stage-specific logs in intermediate directories

### Debug Mode

Run with verbose output:
```bash
process_hcp_asl --verbose [other arguments]
```

### Getting Help

1. Check log files for specific error messages
2. Verify environment setup and dependencies
3. Test with minimal dataset first
4. Consult FSL documentation for tool-specific issues
5. Report bugs to [GitHub Issues](https://github.com/physimals/hcp-asl/issues)

## Advanced Usage

### Custom Empirical Banding Factors

Use custom banding correction factors:
```bash
process_hcp_asl \
  [other arguments] \
  --eb_factors /path/to/custom_banding_factors.txt
```

### Longitudinal Processing

Process longitudinal data:
```bash
process_hcp_asl \
  [other arguments] \
  --longitudinal \
  --longitudinal_template TEMPLATE_SESSION
```

### Custom Output Directory

Change output directory name:
```bash
process_hcp_asl \
  [other arguments] \
  --outdir custom_asl_results
```

### Performance Optimization

For large datasets or high-performance computing:

1. **Parallel Processing**: Use `--cores N` to match available CPU cores
2. **Stage Parallelization**: Run different subjects on different stages simultaneously  
3. **Storage**: Use fast local storage for intermediate files
4. **Memory**: Ensure adequate RAM (16GB+ recommended)

### Integration with HCP Pipelines

The pipeline integrates with standard HCP processing:

1. **Prerequisites**: Run HCP Structural and fMRI pipelines first
2. **Shared Data**: Uses T1w, wmparc, and ribbon from structural pipeline  
3. **Output Format**: Produces HCP-compatible CIFTI outputs
4. **Directory Structure**: Follows HCP conventions

This comprehensive guide should help you successfully process ASL data with the HCP pipeline. For additional support, consult the pipeline documentation and FSL resources.
