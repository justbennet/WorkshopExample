# AFNI 
This is an example of running Analysis of Functional Neuroimages (AFNI)
software to preprocess data for resting state analysis. This analysis
pipeline requires a resting state functional connectivity file and a
high resolution structural T1 image.

Overview. Once you have set up your subject directory, you use the command
`afni_proc.py` to create a generated processing script that does the actual
preprocessing. In a normal workstation environment you would just run this
processing script. On Flux, you would create and submit a small batch shell
script that runs this processing script on the cluster.

## Step 1. Load modules

To access the AFNI software, you need to load it into your environment
so that the commands are in your path. To do this, use the following
command (the `$` indicates the prompt; type what comes after it).

	$ module load afni
   
You can see what modules are loaded, and what version of AFNI you are running,
by listing loaded modules.

	$ module list


## Step 2. Set up data

First put your data into a subdirectory in your allocation on `/scratch`.
In my case, space is allocated for me in `/scratch/nbohnen_flux/shared`.
I will create a directory called `/scratch/nbohnen_flux/shared/subj01`
and place my structural file `T1.nii.gz` and my functional file `rest.nii.gz`
in that subject directory. 

## Step 3. Generate the processing script

The command `afni_proc.py` has many options for generating different processing
streams. This example is based on their Example 9c, for resting state analysis
with ANATICOR, which regresses out signal for each voxel from the local
white matter. One option that you should not use is `-execute`, which executes
the generated processing script right away. Instead, you want to save it and
run it on the compute nodes of the cluster.

This command is in the script `run_afni_procpy`. 

```
afni_proc.py -subj_id subj1 \
   	-dsets rest.nii.gz \
   	-copy_anat T1.nii.gz \
   	-blocks despike tshift align tlrc volreg blur mask regress \
   	-volreg_align_e2a \
   	-volreg_tlrc_warp \
   	-volreg_warp_dxyz 2 \
   	-align_opts_aea -cost lpc+ZZ \
   	-tlrc_base MNI152_T1_2009c+tlrc \
   	-volreg_align_to MIN_OUTLIER \
   	-regress_anaticor \
   	-regress_censor_motion .25 \
   	-regress_censor_outliers .1 \
   	-regress_bandpass 0 .2 \
   	-regress_apply_mot_types demean deriv \
   	-regress_est_blur_epits
```
   	
When you run this command, you will generate a script called `proc.subj1`. 

### Step 4. Create a batch file to run the processing script

An example batch script is in `subj1.pbs` (Need to describe what to change, or override)

Results will be saved in the directory `subj1.results`.
