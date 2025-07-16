# ROI-based analysis and voxel-based asymmetry

ROI summary statistics in Stage 10: “Summary statistics within ROIs.” This uses FSL’s oxford_asl_roi_stats.py to produce CSV tables 
and text files of mean perfusion/ATT in various regions (global gray matter `*_gm_mean.txt`, white matter `*_wm_mean.txt`, and `roi_stats.csv`)

In the main pipeline code (scripts/run_pipeline.py), Stage 10 is executed (`oxford_asl_roi_stats.py` which generates the ROI stats files, 
we add a call to a new function (`compute_asymmetry_map`) that will generate the left–right asymmetry volume. 

?? (do we have these images?) We need to provide the necessary inputs (the perfusion images and brain masks in T1 space) for this new computation. 
?? (do we now have atlases?) The ROI CSVs will still list left vs. right region averages if an atlas is used.
 
