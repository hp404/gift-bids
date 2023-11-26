# PET Preprocessing for GIFT BIDS-App
### Pipeline before GIFT ICA using PetSurfer and PETPrepMATLAB
![TReNDS](https://trendscenter.org/wp-content/uploads/2019/06/background_eeg_1.jpg)
### Short Description of PET to GIFT Pipeline
This pipeline assumes PET data (a raw PET image and an anatomical T1 MRI in BIDS format). It will then use PETprep_HMC (https://github.com/mnoergaard/petprep_hmc), PETPrepMATLAB (https://github.com/mnoergaard/petprepMATLAB), PetSurfer (https://surfer.nmr.mgh.harvard.edu/fswiki/PetSurfer) and a small BASH script to normalize PET images for GIFT independent component analysis processing. This pipeline assumes the PET data is collected in 4 frames per subject. Check the next section for more details.
### PET to GIFT Pipeline
The following is an ouline of the different steps used to process PET for GIFT.
1. Example of data selection. This example was first run after selecting an ADNI dataset for control subjects, having florbetapir (FBP) PET tracer collected in 4 frames, all four being 5 minutes long. Since 3T MRI T1 images are abundant in the ADNI dataset, all subjects with FBP images, the 3T T1 MRI, closest in time, were selected to pair the subject's FBP image. 
2. Formatted ADNI data into BIDS format according with following:
    1. /myfiles/bidsRoot
        1. participants.tsv
        2. participants.json
        3.	other files
        4.	derivatives
            1. freesurfer
                1. sub-xxxx
        5.	sub-xxxx
            1. ses-baseline
                1. pet
                    1. sub-xxxx_ses-baseline_pet.nii
                    2. sub-xxxx_ses-baseline_pet.json
3. Using FreeSurfer and its recon-all function to process 3T T1 MRI to create a reference frame for the FBP images according with following (after placing t1s at /myfiles/fs/sub-xxxx.nii):
    1. recon-all -subjid sub-xxxx -i /myfiles/fs/sub-xxxx.nii -all -sd /myfiles/bidsRoot/derivatives/freesurfer/sub-xxxx
4. After BIDS data was placed under /myfiles/bidsRoot the Open NeuroPET software PETprep_HMC (head motion correction) was run, correcting the 4 PET frames for head motion, using following syntax:
    1. ```
python3 run.py --bids_dir /myfiles/bidsRoot/ --output_dir /myfiles/bidsRoot/derivatives/petprep_hmc/ --n_procs 7 --analysis_level participant  --participant_label 0001 0002 ... 00NN
```
        1. , where 0001 0002 ... 00NN, represents the subject IDs that will be run.
5. Using the motion correction from step 4, and FreeSurfer results from step 3, PETprepMATLAB was run, which basically is a wrapper for petsurfer, getting properties of the cerebellar cortex size and intensities used to calculate SUVR, according with following steps:
    1. In the file “PETPrep.m”, comment out the line having syntax (since freesurfer was already run): ReconAll(BIDS) 
    2. Change config.json with the lines having (since petrep_hmc was already run):
       "mc": {"precomp": "", "cost": "normcorr",  "dof": 6,  "save_plots": true, "refvol": 8},
to:
        {"mc": {"precomp": "hmc_workflow" },

6. [Toolboxes](#secTools)
	1. [Mancovan](#secToolMan)
