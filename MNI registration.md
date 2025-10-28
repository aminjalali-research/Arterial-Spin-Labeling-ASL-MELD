Does hcp_asl pipeline produce a perfusion map in only subject T1 space or subject T1 space + MNI template space?

Review asymmetry index pipelines in more detail - what template did they register to? how did they deal with symmetry?

Perfusion Maps Produced:
Subject T1w Space:
```
1. Subject T1w Space:

T1w/ASL/perfusion_calib.nii.gz - Primary perfusion map
T1w/ASL/pvcorr_perfusion_gm_calib_masked.nii.gz - PV-corrected GM
T1w/ASL/pvcorr_perfusion_wm_calib_masked.nii.gz - PV-corrected WM

2. MNI Template Space:

MNINonLinear/ASL/perfusion_estimation/std_space/perfusion_calib.nii.gz
MNINonLinear/ASL/perfusion_estimation/std_space/perfusion_var_calib.nii.gz 
MNINonLinear/ASL/perfusion_calib.nii.gz - Main MNI output
MNINonLinear/ASL/perfusion_calib_Atlas.dscalar.nii - CIFTI format
```

Pipeline produces separate perfusion maps rather than combinations:
```
T1w space: Located in {subject}/T1w/ASL/ directory (copied in stage 12)
MNI space: Located in {subject}/MNINonLinear/ASL/ directory (warped in stage 12)
Implementation: In key_outputs.py, the copy_key_outputs() function handles transformation using results_to_mni_asl script with FSL warping tools
Format: Separate NIfTI files for each space, not combined 

The exact code locations are:
T1w processing: run_pipeline.py lines 450-600 (stages 6-9)
MNI warping: run_pipeline.py stage 12 + key_outputs.py lines 1-200
```

## Asymmetry Analysis Context:
HCP ASL Pipeline Limitations for Asymmetry:
❌ Current Issues:

- No symmetrical template option - Uses standard MNI152 only
- No voxel-based asymmetry analysis implemented
- Limited ROI framework - Only 3 vascular territories vs ASLPrep's 15+ atlases
- No hemisphere-specific processing

----
Recommended Asymmetry Approach (Based on ASLPrep):

```
1. Template Registration Strategy:

- MNI152NLin6Asym - Used for atlases (contains asymmetric features)
- MNI152NLin2009cAsym - Primary template for QC and analysis

2. Multi-Atlas Asymmetry Analysis: Implement 4S, Glasser, Gordon atlases with bilateral ROI analysis and calculate laterality indices for each atlas region.

- 4S Atlas: 156-1056 parcels with hemisphere-specific regions
- Glasser Atlas: 360 parcels with L/R designation
- Gordon Atlas: 333 parcels with bilateral coverage
- HCP CIFTI: Separate cortical surface processing per hemisphere
```

ASLPrep includes atlases across 3 categories:
```
Combined Cortical/Subcortical (4S Series - 10 atlases):
- 4S156Parcels through 4S1056Parcels (10 resolutions: 156, 256, 356, 456, 556, 656, 756, 856, 956, 1056 parcels)
- Combines Schaefer 2018 cortical atlas + CIT168 subcortical + Diedrichson cerebellar + HCP thalamic + HCP amygdala/hippocampus

Cortical-Only (4 atlases):
- Glasser (HCP Multi-Modal Parcellation) - 360 parcels
- Gordon (Surface-based parcellation) - 333 parcels
- Plus Glasser and Gordon variants

Subcortical-Only (3 atlases):
- Tian (subcortical atlas) - thalamic/striatal regions
- HCP (CIFTI subcortical parcellation)
- Plus variants

Location: atlas.py lines 4-46 defines the complete atlas framework
```

# Symmetrical Template Analysis
ASLPrep template Spaces:

- Primary: MNI152NLin2009cAsym (asymmetric template)
- Atlas space: MNI152NLin6Asym (also asymmetric)
Processing: Complex warping chain: MNI152NLin6Asym → MNI152NLin2009cAsym → T1w → ASL
Templates are NOT symmetrical (both use "Asym" designation)
Solution: However, ASLPrep provides hemisphere-specific processing with hemi-L and hemi-R outputs

---
















