# neuro readme
- Uses CamCAN data of 606 participants with power resolution of 0.5
- Currently uses non-continuous definitions of bands from Brainstorm
- BEFORE Averaging: each subject (row) holds power readings of 0-149.5 (300 total) for each ROI (68 total)
-  size = 606 x 20800
 - AFTER Averaging: each subject (row) holds a power reading for each (band x ROI) combination (408 total)
    => size = 606 x 408

 - The CamCAN dataset values are extremely different from the epilepsy values so this is corrected in 2 ways
    Option #1: Z-score
        => First convert [Subject, PSDxROI] to [Subject, PSD, ROI]
        => For each subject:
            For each reading of the PSD, subtract the average value for that band across all ROIs
        => Each reading of a PSD is now like a Z-score

        => dfAvg3 contains the Z-scored healthy controls
        => df_epilepsy contains the Z-scored patient values

    Option #2: Normalization
        => For each subject:
            For each reading of the PSD, divide it by the total sum of all bands across all ROIs (aka the sum of all features)
        => Each reading is now normalized to the total power of a subject

        => dfAvg4 contains the normalized healthy controls
        => df_epilepsy2 contains the normalized patient values

 - Patient data are expected to be already grouped into the 68 ROIs according to the DK atlas
    => Each row is a ROI and each column is a band
        => size = 68 x 6

 - Measuring abnormality using basic 0-6 scoring
    => use is_statistically_significant_no_model
    => if a patient feature value lies outside the 2.5 and 97.5 percentiles of the healthy controls, it gets a score of 1

 - Measuring abnormality using detailed scoring
    => use is_statistically_significant_no_model_dist
    => if a patient feature value lies outside the 2.5 and 97.5 percentiles of healthy controls,
        calculate the absolute distance to its closest bound
    example A: 2.5th = 0.5, 97.5th = 5.0. Patient feature value = 0.2. Abnormality = 0.5 - 0.2 = 0.3
    example B: 2.5th = 0.5, 97.5th = 5.0. Patient feature value = 7.1. Abnormality = 7.1 - 5.0 = 2.1
    => this is meant to determine tie-breakers between ROIs which are equally abnormal under the 0-6 scoring

 - Measuring abnormality per ROI vs per feature
    => the main goal is to interpret abnormality on a spatial basis but there is a per-feature version of
        functions to investigate abnormality further

 - Mapping abnormality values to a cortex model
    => The mapping code lives inside the MATLAB script (https://github.com/dutchconnectomelab/Simple-Brain-Plot)
        => I've added my script, but you have to download all helper files from the Github
    => Unfortunately, the order of ROIs in the script differs from the order used in the Python script
    => You have to manually map the values to the order used in the MATLAB script
    => In the MATLAB script, the DK atlas has 82 entries instead of 68
        The first 14 entries are not displayed BUT are still considered when creating the colourmap range
        SOLUTION: set the value of the first 14 entries to either the min or max of your deviance values
        -> as long as those entries are within the bounds of your actual abnormality values, they won't affect the colourmap

 ******
 Careful about which variables you're using. If you want to adjust your values using option #1, you need to use dfAvg3 and
 df_epilepsy in testing abnormality functions. If you're using option #2, you must use dfAvg4 and df_epilepsy2 in testing.
 I hope this code is helpful <3
