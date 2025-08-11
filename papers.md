# Paper A. Arterial spin labelling perfusion MRI analysis for the Human Connectome Project Lifespan Ageing and Development studies.

## 1. CIFTI Format and Grayordinates
What it is: A unified coordinate system combining cortical surface vertices and subcortical volume voxels into "grayordinates."

Why we do this: for space efficiency! Reduces storage by 84% compared to storing full brain volumes.
Gray matter on the cortex is a 2D sheet (better analyzed as a surface), while subcortical structures are truly 3D
Better alignment: Surface-based registration aligns cortical areas better than volume-based methods.

## 2. Gradient Nonlinearity Correction
What it is: Correcting spatial distortions caused by imperfect magnetic field gradients.

Why we do this: MRI scanners assume perfectly linear gradients, but they're actually slightly curved.
Objects appear stretched or compressed, especially far from the scanner's center; we need accurate anatomy before any analysis.

## 3. Two-Stage Registration Strategy
What it is: Creating both a "native space" (individual's true brain size) and "MNI space" (standard atlas).
Why we do this: Native space: Preserves true brain dimensions for measuring volumes and doing tractography.
MNI space: Enables comparison across subjects and studies.



