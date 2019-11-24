# SPM UP

SPM utlities plus are tools designed to get the best of mass univariate analyses.

## QA

The QA folder contains a series of tools to check the quality of your images. Some metrics can be used at the group level as covariates, which can be important when comparing different group of subjects, if their quality metrics are different.

The basic usage is to call spmup_anatQA.m spmup_temporalSNR.m spmup_fisrt_level_qa.m spmup_second_level_qa.m.

### SNR and beyond

_spmup_anatQA:_

Inspired by the [Preprocessed Connectome Project Quality Assurance Protocol](http://preprocessed-connectomes-project.org/quality-assessment-protocol/), this function takes an anaotmical image along with gray (c1) and white (c2) matter images to returns
- SNR (Signal-to-Noise Ratio, the mean intensity within gray and white matter divided by the standard deviation of the values outside the brain),
- CNR (Contrast to Noise Ratio, the mean of the white matter intensity values minus the mean of the gray matter intensity values divided by the standard deviation of the values outside the brain),
- FBER (Foreground to Background Energy Ratio, the variance of voxels in grey and white matter divided by the variance of voxels outside the brain),
- EFC (Entropy Focus Criterion, the entropy of voxel intensities proportional to the maximum possibly entropy for a similarly sized image, indicates ghosting and head motion-induced blurring),
- an attempt on a rough asymetry measure (Left-Right/Left+Right Assuming the [0,0,0] coordinate is roughly in the middle of the image - large difference can indicate an issue with the image (for healthy brains).

_spmup_temporalSNR:_

This function recapitulates tSNR as described in [Thomas Liu (2016)](https://www.sciencedirect.com/science/article/abs/pii/S1053811916304694) and the physio to thermal noise ratio and correlation discussed in [Wald and Polimeni (2017)](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5483395/). The function is retuning:
- tSNR.GM: mean GM signal / std over time (estimate BOLD from GM>(WM+CSF))
- tSNR.WM:  mean WM signal / std over time (estimate non-BOLD from WM>(GM+CSF))
- tSNR.CSF: mean CSF signal / std over time (estimate non-BOLD from CSF>(GM+WM))
- tSNR.Background:  mean signal outside mask (GM+WM+CSF) / std over time + report the data as this should only be termal noise, i.e. gaussian distributed (figure print)
- tSNR.average (tSNR): mean signal / sqrt(std(GM)^2+std(WM+CSF)^2+std(Background)^2)
- tSNR.image (SNR0):  mean signal inside mask / std outside mask over time
- tSNR.physio2termal_ratio: sqrt((tSNR(whole image)/SNR0(brain only))^2-1)
- tSNR.physio2termal_corr: correlation between images
- tSNR.roi: tSNR for increased ROI (from in mask by increasing slices) ~linear function of srqrt(nb voxels)
- tSNR.signal_mean: sqrt(std(GM)^2+std(WM+CSF)^2) / sqrt((SNR0^2/tSNR- 1)/SNR0^2)
- a tSMR_time_series.nii image is also saved on the drive, showing tSNR in each voxel for GM, WM and CSF as computed above
- a post-scrip figure with plots of noise, tSNR and ratio with SNR0, noise per increasing ROI size (should be a line)

_spmup_sfs_:

This function computes the Signal Fluctuation Sensitivity metric [DeDora et 2016](http://journal.frontiersin.org/article/10.3389/fnins.2016.00180/full) which is arguably or more sensitive metric than tSNR to mesure BOLD variations in resting state fMRI. By default sfs os returned in the 7 resting state networks from [Yeo et al 2011](https://www.physiology.org/doi/full/10.1152/jn.00338.2011).

### 1st level QA

_spmup_fisrt_level_qa_

This high level functions returns information on files generated by other functions:
- spm_basics: mean std images are correlated
- spmup_realign_QA: makes plots of motions and globals, creates an augmented design matrix (calling spmup_censoring) and makes movies

_spmup_censoring:_

This function takes motion parameters along with any valued vectors to create a design.txt file to be used as regressors. Along with motion parameters (and Voltrera expansion if specified), a series of binary vectors are added for outlying data. Values vectors would typically come from spmup_FD and spm_globals. Note that outliers will differ from the 'Time series outliers' functions detailled below, because within spmup_censoring outliers are defined with high specificity but low sensitivity using S-outliers i.e. less outliers are found but they are for sure outliers, while the default of those functions is low specificity but high sensitivity.

### 2nd level QA

_spmup_second_level_qa_

This function takes a group level SPM.mat to then check for signal droupout from individual masks (spmup_find_dropout)


### Time series outliers

There are many ways to find outlying volumes in fMRI, either based on the volumes themselves or based on derived motion parameters. These are particularly useful to measure/see the effect of preprocessing. Each of these function returms vector values and logical vectors indicating outliers. Vector values can be used in spmup_censoring with uses a less sensitive metric to define outiers. Here, outliers are defined with high sensitivity but low specificity using Carling's K i.e. some declared outliers might not actually have artefacts (see spmup_comp_robust_outliers) but this is whart is needed to diagnose problems. These functions are intented to measure/see the effect of preprocessing (i.e. run on raw then on reprocessed).

_spmup_voxel_outliers:_

This function is similar to [3dToutcount](http://afni.nimh.nih.gov/pub/dist/doc/program_help/3dToutcount.html) retuning which volumes are outliers based on the number of outlying voxels (defined as time points deviating from the median).

_spmup_volumecorr:_

This function computes the volume-wise correlations and also returns outlying volumes. Data are expected to be correlated in time, a large shift in the overall correlation indicates an issue.

_spmup_spatialcorr:_

This function compute the slice-wise correlations. This is performed both between volumes (thus similar to spmup_volumecorr) and within volumes. This returns a figure useful to spot spatial dropouts.

_spmup_FD:_

This function computes the framewise displacement from motion parameters defined as the L1 and L2 norms from derivatives, and returns outliers.
