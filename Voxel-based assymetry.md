# Left–Right Lobe Asymmetry Analysis in ASL Perfusion MRI

Overview: Brain perfusion asymmetry (left vs. right hemisphere or lobes) can be assessed at different scales. Below we compare voxel-based, ROI-based, and surface-based methods for analyzing asymmetry in arterial spin labeling (ASL) perfusion MRI. For each, we explain how asymmetries are quantified, provide any relevant formula with asymmetry indices (AI) with attention to sensitivity, spatial resolution, and robustness. 

1. Region-of-Interest (ROI) volumetry: Segment each anatomical lobe or nucleus, add up all voxels on each side, and compare the two scalars.
It compare perfusion values aggregated over predefined regions on the left and right, such as lobes. It takes the average CBF in the entire left frontal lobe and right frontal lobe and computes a difference. ROI methods reduce the problem to comparing a few summary values (one per region per side) instead of every voxel.

Toolkits like FSL provide parcellations: one can use an atlas to extract left/right ROI mean CBF and then compute asymmetry. 

```python
import nibabel as nib
import numpy as np

# 1. Load T1 and labels
img = nib.load('T1.nii.gz')
labels = nib.load('labels_LR.nii.gz')  # ROI mask with left/right-coded regions

data = img.get_fdata()
labs = labels.get_fdata()

# 2. For each ROI pair compute volumes
def roi_volume(label_idx_left, label_idx_right):
    V_L = np.sum(data[labs == label_idx_left] > 0)
    V_R = np.sum(data[labs == label_idx_right] > 0)
    return (V_L - V_R) / ((V_L + V_R) / 2)

# example
print("Temporal lobe AI:", roi_volume(17, 53))
```

2. Voxel-Based Asymmetry: Voxel-wise methods compute asymmetry at each voxel by comparing perfusion in a location on the left vs. the corresponding location on the right hemisphere. This typically involves registering the brain to a symmetric template or directly flipping one hemispheric image and subtracting it from the other. The result is an asymmetry index map highlighting where perfusion differs between hemispheres
- AI_voxel(x) = (I(x) − I_mirror(x)) / ((I(x) + I_mirror(x))/2) (computed on modulated gray- or white-matter probability maps).

Voxel-based analyses offer high spatial resolution. They can detect focal asymmetries at the level of a few millimeters, without being constrained to large predetermined areas. This fine-grained approach is objective and exhaustive, surveying the whole brain for even subtle perfusion differences. It is especially useful when asymmetries are localized (e.g. a small cortical or subcortical region with reduced CBF) that ROI averaging might miss.

3. Surface-Based Morphometry (SBM)– using FreeSurfer: Vertex-wise cortical thickness, surface area, gyrification.
Surface-based analysis involves projecting perfusion data onto the cortical surface and leveraging cortical topology to assess asymmetry.
The intuitive idea is that surface-based methods respect the geometry of the cortex, possibly allowing a more precise left-right comparison of cortical perfusion than voxel-based volume mapping.

Gyral patterns are matched on the sphere, reducing blurring across sulci that can happen with volume smoothing. This often means surface-based methods can detect differences that volume-based (voxel) methods might smooth out or miss. In fact, a recent study reported that surface-based analysis captured greater changes in perfusion between patients and controls than voxel-based analysis on the same data.

Paper: Associations of Cerebral Blood Flow Patterns With Gray and White Matter Structure in Patients With Temporal Lobe Epilepsy 
```bash
recon-all -i T1.nii.gz -subject sub01 -all
# Surfaces will include lh.thickness, rh.thickness
```
```python
import numpy as np
thick_l = np.loadtxt('sub01/surf/lh.thickness')
thick_r = np.loadtxt('sub01/surf/rh.thickness')[mirror_idx]
AI_vertex = (thick_l - thick_r) / ((thick_l + thick_r)/2)
```
Limitation: Surface-based analysis is naturally limited to cortical structures; perfusion asymmetries in deep brain (basal ganglia, thalamus, etc.) require other methods or projecting those to subcortical surfaces/ROIs. So it’s not a full replacement if subcortical perfusion is of interest.

4. Tensor-/Deformation-Based Morphometry (TBM/DBM): After registering to a symmetric template, compute the Jacobian determinant J(x) that tells local expansion/atrophy, then compare J vs. J_mirror.
- AI_TBM(x) = log J(x) − log J_mirror(x) or equivalently 2 × (J − J_mirror)/(J + J_mirror)

5. Diffusion-Tensor / White-Matter Asymmetry: Calculate Fractional anisotropy (FA) and mean diffusivity (MD) skeleton.
```python
FA_L = fa_skeleton[mask_L]
FA_R = fa_skeleton[mask_R_mirrored]
LI_FA = 2 * (FA_L - FA_R) / (FA_L + FA_R)
```

6. Intensity-Driven Left–Right Intensity Asymmetry (LRIA): Detect scanner-induced or pathological T1/T2-FLAIR signal shifts. 
```python
I_L = np.mean(data[labs == left_roi])
I_R = np.mean(data[labs == right_roi])
LRIA = 100 * (I_L - I_R) / ((I_L + I_R) / 2)
```

7. Shape-Specific Analyses: Represent each 3-D structure (hippocampus, amygdala…) as a set of spherical-harmonic coefficients; compare coefficient vectors or radial distance maps after mirroring.
- Shape-AI = mean|r_L(θ,φ) − r_R(θ,φ)| / mean(r_avg)

8. Functional MRI Laterality Index
- LI = (LH − RH)/(LH + RH); many variants exist (e.g., Dice/Jaccard-based)

-------------------
| Method        | Asymmetry Measure<br/>[Source] | Spatial Scale | Sensitivity<br/>(to subtle differences) | Robustness<br/>(noise, artifacts) | Notable Strengths | Key Limitations |
|---------------|---------------------------------------------------------------------|--------------|--------------------------------------|-----------------------------------|-------------------|----------------|
| **Voxel-based**   | Voxel-wise AI map (e.g. `$(L-R)/((L+R)/2)$)`<br/>[dbm.neuro.uni-jena.de](https://dbm.neuro.uni-jena.de) | High (millimeter voxels)<br/>[dbm.neuro.uni-jena.de](https://dbm.neuro.uni-jena.de) | High: detects small focal asymmetries (e.g. tiny cortical or subcortical perfusion deficits)<br/>[dbm.neuro.uni-jena.de](https://dbm.neuro.uni-jena.de) | Lower: sensitive to noise & misregistration; requires smoothing & careful stats<br/>[dbm.neuro.uni-jena.de](https://dbm.neuro.uni-jena.de) | – Unbiased, whole-brain search for asymmetry (no a priori ROI)<br/>– Generates detailed asymmetry maps for visualization<br/>[pubmed.ncbi.nlm.nih.gov](https://pubmed.ncbi.nlm.nih.gov)<br/>– Proven high detection of subtle lesions (e.g. improved epilepsy focus localization) | – Prone to false positives if data quality is poor (must correct for multiple comparisons)<br/>– Assumes perfect left-right alignment; anatomical differences can confound results without symmetric normalization<br/>[dbm.neuro.uni-jena.de](https://dbm.neuro.uni-jena.de)<br/>– More complex analysis pipeline |
| **ROI-based**     | ROI laterality index or ratio (e.g. `$L/R$` or `$(L-R)/(L+R)$`)<br/>[pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov) | Moderate/Low (cm-scale regions)<br/>[dbm.neuro.uni-jena.de](https://dbm.neuro.uni-jena.de) | Moderate: will catch large or region-wide asymmetries, but may miss very localized ones<br/>[dbm.neuro.uni-jena.de](https://dbm.neuro.uni-jena.de) | High: averaging reduces noise; results are reliable and repeatable<br/>[pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov) | – Simple to compute and interpret (gives a single number per region, e.g. “80% perfusion on left vs right”)<br/>– Little preprocessing needed; robust even on single-case basis (common in clinical reports)<br/>[pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov) | – Coarse resolution: can mask intra-ROI heterogeneity (small affected area diluted by normal tissue)<br/>– Dependent on ROI definition; requires that the chosen regions correspond to functional units of asymmetry<br/>– Limited number of measurements (may overlook unexpected asymmetry outside defined ROIs) |
| **Surface-based** | Vertex-wise asymmetry (left vs. right cortex), often on a symmetric surface template<br/>[pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov) | High along cortex (sub-millimeter on surface mesh) | High (cortex): very sensitive to cortical perfusion differences, preserving gyral pattern detail<br/>[pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov) | High (cortex): avoids cross-sulcus blurring; cortical alignment improves signal consistency<br/>[pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov) | – Respects cortical anatomy, yielding sharper and more accurate localization of asymmetry than volume methods<br/>– Can integrate with cortical metrics (e.g. map perfusion asymmetry alongside cortical thickness or function)<br/>– Shows improved detection of cortical hypoperfusion vs. voxel-based in some studies<br/>[pmc.ncbi.nlm.nih.gov](https://pmc.ncbi.nlm.nih.gov) | – Focuses on cortex: asymmetry in deep structures needs separate treatment<br/>– Requires good quality structural MRI and segmentation; errors in surface extraction can affect results<br/>– Not as widely used clinically; interpretation needs familiarity with surface maps |


```python
"""
Assumptions:
- ASL CBF map is preprocessed (coregistered, normalized, skull-stripped, same orientation for L/R)
- Standard atlases or surfaces are available for ROI and surface-based analyses
- Uses NiBabel, Nilearn, Numpy, Nibabel, FreeSurfer, and optionally, BrainSpace or SurfStat (for surfaces)
- For simplicity, minimal plotting is used (for illustration)
"""

import numpy as np
import nibabel as nib
import matplotlib.pyplot as plt
from nilearn import plotting, image, datasets, surface
from nilearn.maskers import NiftiLabelsMasker
from nilearn.surface import vol_to_surf

# ----------------------
# INPUTS
# ----------------------
# Path to preprocessed ASL CBF map (NIfTI)
asl_path = 'asl_cbf_map.nii.gz'  # (replace with your path)
# Atlas with left/right ROI labels (e.g. Harvard-Oxford, AAL)
atlas_path = 'atlas_labels.nii.gz'  # (replace with your path)
# List of left-right ROI pairs (by atlas index)
left_rois = [1,3,5,7]   # Example ROI label indices (even=left, odd=right)
right_rois = [2,4,6,8]

# For surface-based: pial/white surfaces, or use fsaverage
fsaverage = datasets.fetch_surf_fsaverage()

# ----------------------
# Load images
# ----------------------
cbf_img = nib.load(asl_path)
atlas_img = nib.load(atlas_path)
cbf_data = cbf_img.get_fdata()
atlas_data = atlas_img.get_fdata().astype(int)

# ----------------------
# 1. ROI-BASED ASYMMETRY
# ----------------------
def roi_asymmetry(cbf_data, atlas_data, left_rois, right_rois):
    ai_list = []
    for l, r in zip(left_rois, right_rois):
        left_mask = (atlas_data == l)
        right_mask = (atlas_data == r)
        left_val = np.mean(cbf_data[left_mask])
        right_val = np.mean(cbf_data[right_mask])
        ai = (left_val - right_val) / ((left_val + right_val) / 2)
        ai_list.append({'left_roi': l, 'right_roi': r,
                        'left_mean': left_val, 'right_mean': right_val,
                        'AI': ai})
    return ai_list

roi_results = roi_asymmetry(cbf_data, atlas_data, left_rois, right_rois)
print("\nROI-Based Asymmetry Results:")
for res in roi_results:
    print(res)

# ----------------------
# 2. VOXEL-BASED ASYMMETRY
# ----------------------
def voxel_asymmetry_map(cbf_data):
    # Assumes data is in RAS+ orientation and midline is at data.shape[0]//2
    x = cbf_data.shape[0]
    left = cbf_data[:x//2, :, :]
    right = cbf_data[x//2:, :, :]
    # Flip right hemisphere to align to left
    right_flipped = np.flip(right, axis=0)
    # Truncate for equal size if odd number
    min_shape = np.min([left.shape, right_flipped.shape], axis=0)
    left_crop = left[:min_shape[0], :min_shape[1], :min_shape[2]]
    right_crop = right_flipped[:min_shape[0], :min_shape[1], :min_shape[2]]
    ai_map = (left_crop - right_crop) / ((left_crop + right_crop) / 2 + 1e-5)
    # Merge back for visualization (left AI, right zeros)
    full_ai = np.zeros_like(cbf_data)
    full_ai[:min_shape[0], :min_shape[1], :min_shape[2]] = ai_map
    return full_ai

ai_map = voxel_asymmetry_map(cbf_data)
nib.save(nib.Nifti1Image(ai_map, cbf_img.affine), 'asl_voxel_asymmetry_map.nii.gz')
print("\nVoxel-based AI map saved as 'asl_voxel_asymmetry_map.nii.gz'.")

# Visualize one slice
plt.imshow(ai_map[:,:,ai_map.shape[2]//2], cmap='seismic', vmin=-1, vmax=1)
plt.title('Voxel-based Asymmetry Index (Mid-slice)')
plt.colorbar(label='AI')
plt.show()

# ----------------------
# 3. SURFACE-BASED ASYMMETRY
# ----------------------
def surface_asymmetry(cbf_img, hemisphere='left'):
    # Project CBF to surface mesh (fsaverage, pial)
    mesh = fsaverage['pial_' + hemisphere]
    surf_cbf = vol_to_surf(cbf_img, mesh)
    # Project flipped hemisphere for AI
    mesh_opposite = fsaverage['pial_right' if hemisphere=='left' else 'pial_left']
    surf_cbf_opp = vol_to_surf(cbf_img, mesh_opposite)
    # Flip to align left-right (reverse vertex order)
    surf_cbf_opp_flip = surf_cbf_opp[::-1]
    ai_surf = (surf_cbf - surf_cbf_opp_flip) / ((surf_cbf + surf_cbf_opp_flip) / 2 + 1e-5)
    return surf_cbf, surf_cbf_opp_flip, ai_surf

surf_cbf_left, surf_cbf_right_flip, ai_surf_left = surface_asymmetry(cbf_img, 'left')

# Plot surface AI map
plotting.plot_surf_stat_map(fsaverage['pial_left'], ai_surf_left, hemi='left', title='Surface Asymmetry (Left)', colorbar=True, cmap='seismic')
plt.show()

# ----------------------
# Comparison summary
# ----------------------
print("\nComparison Summary:")
print("- ROI-based: Gives one AI value per region (robust, interpretable)")
print("- Voxel-based: AI map highlights focal asymmetries (high-res, noise sensitive)")
print("- Surface-based: AI per vertex (high-res along cortex, anatomical specificity)")

# (Optional) Save results, generate summary plots, or statistical tests as needed
```




  
