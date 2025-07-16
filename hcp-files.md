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
import nibabel as nib
import numpy as np

def compute_asymmetry_map(perf_path, mask_path, out_path):
    img = nib.load(perf_path)
    data = img.get_fdata()
    brain_mask = nib.load(mask_path).get_fdata() if mask_path else None
    # Flip the perfusion data along the X axis (left-right flip)
    flipped_data = data[::-1, :, :].copy()  
    # Calculate asymmetry index: (L - R) / ((L + R)/2)
    num = data - flipped_data
    den = (data + flipped_data) / 2.0
    # Avoid divide-by-zero by masking or adding epsilon
    asym_index = np.zeros_like(data)
    valid = (den != 0)  # or use brain_mask > 0 for valid voxels
    asym_index[valid] = num[valid] / den[valid]
    # Apply brain mask to clean outside voxels
    if brain_mask is not None:
        asym_index *= (brain_mask > 0)
    asym_img = nib.Nifti1Image(asym_index, img.affine, img.header)
    nib.save(asym_img, out_path)

# ... In the pipeline execution flow (after ROI stats) ...
perf_file = os.path.join(asl_dir, "perfusion_calib.nii.gz")
mask_file = os.path.join(asl_dir, "reg", "ASL_grid_T1w_brain_mask.nii.gz")
out_file  = os.path.join(asl_dir, "perfusion_asym_index.nii.gz")
compute_asymmetry_map(perf_file, mask_file, out_file)
```

