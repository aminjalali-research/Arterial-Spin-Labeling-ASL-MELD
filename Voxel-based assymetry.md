# Left–Right Lobe Asymmetry Analysis in ASL Perfusion MRI

Overview: Brain perfusion asymmetry (left vs. right hemisphere or lobes) can be assessed at different scales. Below we compare voxel-based, ROI-based, and surface-based methods for analyzing asymmetry in arterial spin labeling (ASL) perfusion MRI. For each, we explain how asymmetries are quantified, provide any relevant formula with asymmetry indices (AI) with attention to sensitivity, spatial resolution, and robustness. 

Pipeline: Preprocess (denoising, bias correction, skull-strip), segmentation, registration, run FreeSurfer, then calculate RIO volumetry.  



1. Region-of-Interest (ROI) volumetry: Segment each anatomical lobe or nucleus, add up all voxels on each side, and compare the two scalars.

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

2. Voxel-Based Asymmetry: Warp every brain to a symmetric template, flip the image left–right, subtract, and test each voxel.
- AI_voxel(x) = (I(x) − I_mirror(x)) / ((I(x) + I_mirror(x))/2) (computed on modulated gray- or white-matter probability maps).


3. Surface-Based Morphometry (SBM)– using FreeSurfer: Vertex-wise cortical thickness, surface area, gyrification.
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







  
