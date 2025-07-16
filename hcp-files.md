# ROI-based analysis and voxel-based asymmetry

ROI summary statistics in Stage 10: “Summary statistics within ROIs.” This uses FSL’s oxford_asl_roi_stats.py to produce CSV tables 
and text files of mean perfusion/ATT in various regions (global gray matter `*_gm_mean.txt`, white matter `*_wm_mean.txt`, and `roi_stats.csv`)

In the main pipeline code (scripts/run_pipeline.py), Stage 10 is executed (`oxford_asl_roi_stats.py` which generates the ROI stats files, 
we add a call to a new function (`compute_asymmetry_map`) that will generate the left–right asymmetry volume. 

?? (do we have these images?) We need to provide the necessary inputs (the perfusion images and brain masks in T1 space) for this new computation. Load the perfusion image in T1w space `perfusion_calib.nii.gz`, which is the final calibrated CBF map in the subject’s structural space. We use NiBabel to load the NIfTI and get a NumPy array of dimensions (X, Y, Z).

?? (do we now have atlases?) The ROI CSVs will still list left vs. right region averages if an atlas is used.

Flip the image left-to-right to obtain the mirror image. Since the data is in ACPC-aligned T1 space, the x-axis corresponds to left–right. 
We can flip the array on the x dimension (index 0) to produce a mirrored volume. For example: flipped_data = data[::-1, :, :] (if the first index is the X axis). Compute the asymmetry index for each voxel: `asym_data = (data - flipped_data) / ((data + flipped_data) / 2)`. This implements `(L - R)/((L+R)/2)`. In code, one should be careful to handle division by zero or very low values: if (L+R) is zero or near-zero (which could happen in background or very low-perfusion areas), this formula would be undefined.

?? In practice, using the brain mask (from the pipeline’s outputs) will already eliminate most zero-background voxels. The pipeline provides a brain mask in T1w space (e.g. ASL_grid_T1w_brain_mask.nii.gz in the reg folder). We multiply the asymmetry map by this mask so that outside-brain values are zeroed out and do not produce spurious ratios. 

Save the asymmetry map as a NIfTI. We can name it, for example, perfusion_asym_index.nii.gz (to denote it’s the normalized asymmetry index) and save it in the subject’s T1w ASL directory alongside other outputs. Use the same affine and header as the input perfusion image when saving via nibabel, so that it aligns exactly with the T1 anatomy.

```python
# Required Python Libraries:
import nibabel as nib   # Already used in pipeline
import numpy as np      # Already used in pipeline
import os               # Already used in pipeline

# Additional recommended library for image processing (optional):
import nilearn.image as nli  # For advanced masking and image processing

# Example function for voxel-based ASL asymmetry calculation:
def compute_asymmetry_index(perf_path, mask_path, out_path):
    # Load perfusion and mask images
    img = nib.load(perf_path)
    data = img.get_fdata()

    mask_img = nib.load(mask_path)
    mask_data = mask_img.get_fdata()

    # Flip perfusion data across the X-axis (left-right)
    flipped_data = data[::-1, :, :].copy()

    # Calculate asymmetry index safely, avoiding division by zero
    numerator = data - flipped_data
    denominator = (data + flipped_data) / 2

    asymmetry_index = np.zeros_like(data)
    valid_voxels = (denominator != 0) & (mask_data > 0)
    asymmetry_index[valid_voxels] = numerator[valid_voxels] / denominator[valid_voxels]

    # Save the asymmetry index image
    asym_img = nib.Nifti1Image(asymmetry_index, img.affine, img.header)
    nib.save(asym_img, out_path)

# Integrate this function into your pipeline script (run_pipeline.py):
# (Assuming paths below are adjusted according to your pipeline structure)
perf_file = os.path.join(asl_dir, "perfusion_calib.nii.gz")
mask_file = os.path.join(asl_dir, "reg", "ASL_grid_T1w_brain_mask.nii.gz")
asymmetry_file = os.path.join(asl_dir, "perfusion_asym_index.nii.gz")

compute_asymmetry_index(perf_file, mask_file, asymmetry_file)

# Ensure Nilearn is installed if used:
# pip install nilearn

```
This will produce a new volume where each voxel’s value indicates the relative perfusion difference between the left and right mirrored locations. Positive values mean left > right, negative means right > left, and zero means symmetric. For example, an asymmetry value of +0.2 would mean the left voxel has ~20% higher CBF than the right voxel, while –0.5 would mean the right side has 50% higher perfusion than the left in that region.

Log ratio: Another metric is the log-transformed ratio $\log(L/R)$, which symmetrizes the distribution and handles scaling (log ratio 0 means symmetry, positive = left > right). This could be computed as $\log((L+\epsilon)/(R+\epsilon))$ for numerical stability. It yields similar information but could be beneficial if perfusion values have a skewed distribution.


-------
# MNI Standard Space
We should warp the asymmetry map to MNI space so that all subjects’ asymmetry maps are in a common reference. Since the structural image already has a known transform to MNI (from the FreeSurfer or ANTs registration used in the HCP pipelines), we can apply that same warp to the asymmetry volume. In practice, this could be done by adding an applywarp (FSL) call or using wb_command -volume-warpfield with the warp that was applied to the perfusion images. The pipeline’s design likely has a warp or premade MNI version of the perfusion from FSL’s oxford_asl (the directory listing shows a std_space output for perfusion).
?? what does FSL and freesurfer do sequentially in this pipeline?

For consistency, we can mirror that: once the asymmetry NIfTI is in T1 space, apply the structural-to-MNI warp to get perfusion_asym_index_MNI.nii.gz in the MNINonLinear/ASL folder. This allows group-level analyses or comparison of asymmetry maps across subjects in standard space. (Note: The asymmetry computation itself should not be done in MNI directly because resampling perfusion to MNI then flipping could introduce interpolation differences between left/right. It’s more accurate to compute asymmetry in native space and then warp the result, which preserves the L–R difference signal.)

-----
# Files considerations
?? The pipeline uses the FSL `oxford_asl` tool internally, called from run_pipeline.py to calculate voxelwise perfusion.
(is it true?) The voxelwise perfusion output is usually saved as perfusion_calib.nii.gz.

Atlas Template Registration:
The pipeline includes atlas templates (e.g., Mutsaerts' vascular territories atlas) for ROI analysis.
Around Stage 10, where oxford_asl_roi_stats.py is invoked.

?? We need to ensure oxford_asl voxelwise perfusion calculation is present `run_pipeline.py`. Ensure the atlas template registration (to anatomical or standard MNI space) is already implemented. If missing, you may need to perform registration. Confirm spatial normalization (e.g., flirt, fnirt, or ANTs) to standard MNI or other template spaces. Ensure atlas alignment within the existing pipeline.


CreateDenseScalarASL.sh: Creates dense scalar files for cortical surface analysis.
mt_estimation_pipeline.py: Estimates empirical banding scaling factors.
PerfusionCIFTIProcessingPipelineASL.sh: Projects voxelwise perfusion results onto the cortical surface and MNI space.
results_to_mni.py: Transforms ASL-gridded T1w-space ASL variables into ASL-gridded MNI-space.
run_pipeline.py: Main script orchestrating pipeline stages from preprocessing to ROI statistics.
se_based.py: Script handling SE-based bias estimation.
SubcorticalProcessingASL.sh: Prepares subcortical metrics for CIFTI generation.
SurfaceSmoothASL.sh: Applies surface smoothing to perfusion data.
VolumetoSurfaceASL.sh: Maps volumetric perfusion data to cortical surfaces.


? Shall we?
To run voxel-based asymmetry analysis:
1. Insert the asymmetry calculation function (provided earlier) into run_pipeline.py after stage 10 (ROI statistics).
2. Ensure outputs:
Asymmetry index (perfusion_asym_index.nii.gz) saved in the T1w space.
Transform this asymmetry index to MNI space (results_to_mni.py can be utilized similarly as done for perfusion).
Project the asymmetry map onto cortical surfaces using VolumetoSurfaceASL.sh and include it in your pipeline outputs.























