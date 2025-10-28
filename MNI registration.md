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

## Asymmetry Analysis Context:
HCP ASL Pipeline Limitations for Asymmetry:
❌ Current Issues:

- No symmetrical template option - Uses standard MNI152 only
- No voxel-based asymmetry analysis implemented
- Limited ROI framework - Only 3 vascular territories vs ASLPrep's 15+ atlases
- No hemisphere-specific processing



ASLPrep's Templates:
- MNI152NLin6Asym - Used for atlases (contains asymmetric features)
- MNI152NLin2009cAsym - Primary template for QC and analysis
- Hemisphere-specific processing: hemi-L and hemi-R surface outputs

Atlas Framework for Asymmetry:

4S Atlas: 156-1056 parcels with hemisphere-specific regions
Glasser Atlas: 360 parcels with L/R designation
Gordon Atlas: 333 parcels with bilateral coverage
HCP CIFTI: Separate cortical surface processing per hemisphere
