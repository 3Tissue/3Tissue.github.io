---
title: 'Basic single-subject pipeline'
description: 'A basic single-subject pipeline, which also serves as a useful tutorial to get started.'
creatortwitter: 'MRtrix3Tissue'
---

## Basic single-subject pipeline

This is a simple basic single-subject pipeline for preprocessing data, estimating 3-tissue response functions, performing 3-tissue CSD, visualisation and tractography. Most analyses, whether single-subject or group studies, will typically perform most of these steps for all subjects involved. This pipeline also serves as a good introduction and tutorial to some of the most common processing steps.

For each command, you can read the documentation at the command line by simply entering the command name on its own. Visit [this page]({% link doc/general.md %}) for more information on general usage.

Note that the help page of most commands also lists the scientific reference(s) for the method(s) offered by it. These references are located all the way at the bottom of the help page (you can scroll down faster using the `Page Down` key or directly jump to the bottom using the `End` key on your keyboard).

The pipeline below assumes you're using MRtrix3Tissue v5.1.0 or later. Visit [this page]({% link doc/install/release.md %}) for more information about releases. 

## Preprocessing and 3-tissue CSD modelling

All steps listed in this section are common to most analyses, whether single-subject or group studies.

### Importing data

As the very first step, it's generally recommended to import your data "just once" ("and for all"), and immediately store it as .mif file(s), the so-called "MRtrix Image Format". Not only does this allow you to check whether the data was read and imported correctly, it'll also make your life far more convenient and less error-prone down the track: complex diffusion MRI related information will be stored directly within the .mif file(s) itself, so if the data is imported well, you'll never have to worry about this again during processing.

`mrconvert` is *the* tool to convert between all sorts of image formats (or extract parts of images or change properties) and enables you to read from e.g. DICOM folders or NIfTI (.nii) images and store an image as a .mif file. 

You can import your diffusion MRI data directly from a DICOM folder as follows:

```
mrconvert MY_DICOM_FOLDER dwi.mif
```

If there are multiple scans in the DICOM folder structure, the command will show you a numbered list of all scans it found and you can select the diffusion MRI scan by entering its number (as shown in the list).

If you also have other images you want to import, it's again a good idea to immediately get this over with. For example, if you have scanned a pair of reverse phase-encoded b=0 images (sometimes also called "blip-up blip-down" images), you could import each of these as follows (assuming they're separate scans):

```
mrconvert MY_DICOM_FOLDER AP.mif

mrconvert MY_DICOM_FOLDER PA.mif
```

...and each time select the correct scan from the list, if multiple scans exist in the folder(s).

If your diffusion MRI data comes as a NIfTI (.nii) image instead, the gradient orientations and b-values will typically be stored in 2 separate files (a "bvecs" and "bvals" file). You can provide these to `mrconvert` like this:

```
mrconvert dataset.nii dwi.mif -fslgrad dataset.bvecs dataset.bvals
```

...and they will be correctly converted and the relevant information will be stored *within* the resulting .mif file for you. Once this is done, you'll only need the `dwi.mif` file to continue. There's no need to keep your "raw" DICOM folder(s) or NIfTI image(s) around, if you have them backed up or stored somewhere else.

### Denoising and unringing

Denoising and Gibbs ringing removal ("unringing") are performed *prior* to any other processing steps: most other processing steps, in particular those that involve regridding of the data, would otherwise invalidate the original properties of the image data that are exploited by `dwidenoise` and `mrdegibbs` at this stage, and would render the result prone to errors.

Denoising is performed as the first step:

```
dwidenoise dwi.mif dwi_denoised.mif
```

Gibbs ringing removal ("unringing") is performed immediately after:

```
mrdegibbs dwi_denoised.mif dwi_denoised_unringed.mif
```

The unringing is performed by `mrdegibbs` in each slice of the 3D volumes, so it needs to know the *slice orientation*. If you have imported your data directly from DICOM folder(s) using MRtrix3Tissue v5.1.0 or later, `mrconvert` might have stored this information in the `.mif` file for you, and then `mrdegibbs` will make use of it automatically. If you want or need to specify it manually, you can do so via the `-axes` option: `-axes 0,1` for *axial slices*, `-axes 0,2` for *coronal slices* and `-axes 1,2` for *sagittal slices*. Check the command line documentation of `mrdegibbs` for more details.

It's useful to check the output after each processing step in `mrview`. This can be facilitated by opening both the input as well as the output of a certain step together in `mrview`, which allows you to quickly jump back and forth between them with the `Page Up` and `Page Down` keys on your keyboard: in this way, you can clearly see what the effect of the step is on your data. For example, you can open your `mrdegibbs` input and output via `mrview dwi_denoised.mif dwi_denoised_unringed.mif`.

### Motion and distortion correction

Motion and distortion correction are performed by the `dwifslpreproc` command. As the name suggests, this command interfaces with the [FSL](https://fsl.fmrib.ox.ac.uk) package for you (so make sure you have installed FSL). Behind the scenes, most of the heavy lifting is done automatically by FSL's [topup](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/topup) and [eddy](https://fsl.fmrib.ox.ac.uk/fsl/fslwiki/eddy) tools, so make sure to cite both the FSL software as well as the relevant papers for the methods.

The simplest scenario is to only correct for motion and eddy current-induced distortions:

```
dwifslpreproc dwi_denoised_unringed.mif dwi_denoised_unringed_preproc.mif -pe_dir AP -rpe_none
```

However, if you have also scanned a pair of reverse phase-encoded b=0 images (sometimes also called "blip-up blip-down" images), you can correct for susceptibility-induced EPI distortions as well. To do so, concatenate the reverse phase-encoded b=0 images and change the `dwifslpreproc` command as follows:

```
mrcat AP.mif PA.mif bzero_pair.mif -axis 3

dwifslpreproc dwi_denoised_unringed.mif dwi_denoised_unringed_preproc.mif -pe_dir AP -rpe_pair -se_epi bzero_pair.mif
```

Note that the `-pe_dir` option is used to tell `dwifslpreproc` the *phase-encoding direction* of the scan: `-pe_dir AP` for *anterior-posterior*, `-pe_dir LR` for *left-right* and `-pe_dir SI` for *superior-inferior*. If you provide a pair of reverse phase-encoded b=0 images as above, concatenate them so the *first* one has the same phase-encoding direction as the main diffusion MRI dataset ("AP" in the example above). Check the command line documentation of `dwifslpreproc` for more details and options for more complicated scenarios.

### Bias field correction

In the 3-tissue pipeline, bias fields are actually corrected for robustly later on, in a step *after* the 3-tissue constrained spherical deconvolution (3-tissue CSD) stage by using the `mtnormalise` command (which also jointly performs global intensity normalisation). However, correcting (*additionally*) for bias fields already at this point in the pipeline can *sometimes* help improve the performance of intermediate steps: especially the brain mask estimation in the next step can be affected (for the better, or worse) by this. You should choose whether to perform this step in function of how well the subsequent brain mask estimation turns out with *or without* this step. Whether bias field correction is run at this stage or not, however, does *not* have a significant impact on the performance of `mtnormalise` later on; so that's not something to worry about in any case.

You can correct for bias fields at this point in the pipeline with the `dwibiascorrect ants` command. As the name suggests, this command interfaces with the [ANTs](http://stnava.github.io/ANTs) package for you (so make sure you have installed ANTs). Behind the scenes, most of the heavy lifting is done automatically by ANTs' [N4BiasFieldCorrection](https://ieeexplore.ieee.org/document/5445030) tool, so make sure to cite both the ANTs software as well as the relevant paper for the method.

The `dwibiascorrect ants` command is used as follows:

```
dwibiascorrect ants dwi_denoised_unringed_preproc.mif dwi_denoised_unringed_preproc_unbiased.mif
```

It's a good idea to check the output and assess whether bias fields have been corrected (up to some extent), without introducing extreme values, e.g. closer to the edge of the field of view. Such extreme values would trip up the brain mask estimation. If you're unsure, simply proceed to the next brain mask estimation step in any case to assess the impact directly.

### Brain mask estimation

A brain mask can be estimated automatically from the diffusion MRI data using `dwi2mask` as follows:

```
dwi2mask dwi_denoised_unringed_preproc_unbiased.mif dwi_mask.mif
```

Brain mask estimation is a difficult challenge, and `dwi2mask` is not perfect for all possible scenarios. As mentioned above, if things don't run well out of the box, it's worthwhile checking whether it's better to *not* run the previous `dwibiascorrect ants` step. Problematic scenarios are those where either the whole brain mask is "completely" wrong (the mask doesn't resemble a brain at all), or more commonly where a "gap" or "hole" exists in the mask, sometimes at the bottom (and deeper into the brain) or the brain stem. Sometimes this can be manually corrected using the ROI editing tools in `mrview`. If, on the other hand, you see *extra* bits "attached" to the brain mask (most commonly: one or two eyeballs, or bits related to sinus cavities), this is *not* per se an issue or something to worry about at all, unless in extreme cases. So the gist is to err on the side of "inclusion": make sure all areas of interest are at least *included* in the mask.

Alternatively, if you can obtain a good brain mask from any other source, e.g. estimated from an anatomical (T1w) image using other external tools, that might of course be an appropriate solution as well. Make sure though, the source (e.g. T1w) image is sufficiently aligned spatially to the diffusion MRI data. The "upsampling or regridding" step later in the pipeline will also deal with regridding masks from other sources to your diffusion MRI voxel grid.

### 3-tissue response function estimation

A robust and fully automated unsupervised method to obtain 3-tissue response functions representing single-fibre white matter (WM), grey matter (GM) and CSF from the data itself, is available as the so-called `dwi2response dhollander` command. In MRtrix3Tissue, this runs the algorithm with the latest improvements to single-fibre WM response function estimation [as introduced at ISMRM 2019](https://www.researchgate.net/publication/331165168_Improved_white_matter_response_function_estimation_for_3-tissue_constrained_spherical_deconvolution). A video of the ISMRM 2019 talk with more information and insights is also available [over here](https://youtu.be/7yPSFgLt8CA).

You can run 3-tissue response function estimation as follows:

```
dwi2response dhollander dwi_denoised_unringed_preproc_unbiased.mif response_wm.txt response_gm.txt response_csf.txt
```

You can inspect the resulting response functions directly in any text editor (they're simple text files), or visually using `shview`. The latter is mostly useful to look at the single-fibre WM response. Use the `left` and `right` arrow keys on your keyboard to navigate between b-values in `shview`. If you're unsure about how to assess all of this, you can also run `dwi2response dhollander` as above, but add the `-voxels response_voxels.mif` option. The `response_voxels.mif` image can then be opened in `mrview` (and overlaid with the diffusion MRI data or an aligned anatomical image for spatial reference). This allows to directly inspect which voxels the algorithm ended up sourcing the 3-tissue response functions from.

Note that, for quantitative *group* studies, a single *unique* set of 3-tissue response functions should be used for all subjects. This can for example be achieved by averaging response functions (per tissue type) across all subjects in the study.

### *Optional*: upsampling or regridding

For some pipelines or applications, you might want to consider upsampling (or more generally regridding) your preprocessed diffusion MRI data. If you do this *before* the 3-tissue constrained spherical deconvolution (3-tissue CSD) step, you'll get a higher quality result. Upsampling is useful if you're after a higher quality visualisation, which reveals finer details. It might also benefit image registration, (population) template construction or even tractography. Do note that upsampling can easily generate very large image file(s), and will result in (sometimes drastically) increased computation times for the 3-tissue CSD step. Remember that voxels are *3-dimensional*: e.g. if you were to upsample all voxel dimensions by a factor 2, you're generating 8 times more voxels! If you're just doing a "quick" test run to see if you can produce good 3-tissue CSD results from your data, you might want to skip this step initially.

As an example, regridding to an isotropic voxel size of 1.5mm x 1.5mm x 1.5mm can be achieved using the `mrgrid` command as follows:

```
mrgrid dwi_denoised_unringed_preproc_unbiased.mif regrid dwi_denoised_unringed_preproc_unbiased_upsampled.mif -voxel 1.5
```

If you regrid your data, you'll want to bring the brain mask along to the new grid. To make sure the brain mask is regridded to the *exact* same voxel grid, you can simply provide your regridded diffusion MRI data via the `-template` option to `mrgrid`. This will also work for brain masks that weren't generated on the original diffusion MRI data voxel grid, e.g. those obtained from a (spatially aligned) T1w image. If the regridding is effectively *upsampling*, you can avoid jagged edges around masks by performing linear interpolation followed by a median filtering of the result. The latter can be achieved using the `maskfilter` command. To perform all of this *at once* (without storing an intermediate image), you an directly "pipe" the output of `mrgrid` into the input of `maskfilter`, by replacing the output in the one command and the input in the other command by a dash ("`-`"), and typing a "pipe" ("`|`") between both commands.

Ultimately, all of this can then be achieved by this single line of commands:

```
mrgrid dwi_mask.mif regrid - -template dwi_denoised_unringed_preproc_unbiased_upsampled.mif -interp linear -datatype bit | maskfilter - median dwi_mask_upsampled.mif
```

Note that, even if you don't upsample or regrid your diffusion MRI data, you can still use the same procedure to regrid a mask obtained from a different source, e.g. a (spatially aligned) T1w image, to the grid of your diffusion MRI data.

### 3-tissue CSD modelling

Your data is now ready for *3-tissue CSD modelling* with the previously obtained 3-tissue response functions, which will result in modelling the diffusion MRI data using *WM-like (FOD), GM-like and CSF-like compartments*. There are 2 different *methods* (or algorithms) available to perform 3-tissue CSD. The choice between both depends on what (part of your) data you intend to perform 3-tissue CSD modelling for.

If you want to perform 3-tissue CSD modelling for *multi-shell* data, this can be achieved using the *multi-shell multi-tissue CSD (MSMT-CSD)* method or algorithm, as follows:

```
dwi2fod msmt_csd dwi_denoised_unringed_preproc_unbiased_upsampled.mif response_wm.txt wmfod.mif response_gm.txt gm.mif response_csf.txt csf.mif -mask dwi_mask_upsampled.mif
```

If you want to perform 3-tissue CSD modelling for *single-shell (+b=0)* data (or a *single-shell + b=0* subset of your data), this can be achieved using the *single-shell 3-tissue CSD (SS3T-CSD)* method or algorithm, as follows:

```
ss3t_csd_beta1 dwi_denoised_unringed_preproc_unbiased_upsampled.mif response_wm.txt wmfod.mif response_gm.txt gm.mif response_csf.txt csf.mif -mask dwi_mask_upsampled.mif
```

For the SS3T-CSD method, the ordering of inputs/outputs is *strict* (WM - GM - CSF, in that order). The SS3T-CSD method generalises relatively well out of the box for a range of data, but it's always worthwhile checking the output well. Check [this page]({% link doc/ss3t-csd.md %}) for more details, and consider [providing feedback]({% link feedback-support.md %}) about your experience with SS3T-CSD!

Finally note that, for quantitative *group* studies, a single *unique* set of 3-tissue response functions should be used for all subjects. This can for example be achieved by averaging response functions (per tissue type) across all subjects in the study.

### 3-tissue bias field and intensity normalisation

As mentioned above, bias field correction can be performed robustly directly on (and informed by) the 3-tissue compartments. This can be achieved using the `mtnormalise` command, as follows:

```
mtnormalise wmfod.mif wmfod_norm.mif gm.mif gm_norm.mif csf.mif csf_norm.mif -mask dwi_mask_upsampled.mif
```

This performs global intensity normalisation as well, making sure that amplitudes can be better (more predictably) relied upon for further processing steps.

For group studies, when 3-tissue CSD is performed using a single *unique* set of 3-tissue response functions, the `mtnormalise` step makes the absolute amplitudes directly comparable between all subjects. As such, this step becomes absolutely crucial, even if bias field correction was already applied earlier in the pipeline using `dwibiascorrect ants`; as `dwibiascorrect ants` does *not* perform *global* intensity normalisation.

This concludes the typical series of processing steps common to most single-subject analyses as well as group studies. At this point, most analyses would continue only with the final `wmfod_norm.mif` (for each subject), for certain (3-tissue) analyses also complemented with the final `gm_norm.mif` and `csf_norm.mif` files. The brain mask (`dwi_mask_upsampled.mif`) will often also still come in handy. All other prior files generated along the pipeline, however, can typically be backed up or otherwise safely "moved out of the way" at this stage.

## Extras

This section follows the typical set of common steps above with a couple of extra steps and also helps to assess the final output visually.

### Visualisation

Directionally-encoded colour (DEC) maps are commonly used to visualise diffusion MRI data. Traditionally, these were computed from the diffusion tensor model (using the principal eigenvector's orientation to determine the colour) and scaled by a fractional anisotropy (FA) map. Due to the (now well-known) limitations of the diffusion tensor model though, both the colours as well as the (FA) intensity can be misleading. In a 3-tissue CSD pipeline, however, you can also compute *FOD-based* DEC maps, where both the colours as well as the intensity are computed directly from the *entire FOD*. See [this work](https://www.researchgate.net/publication/276412466_Time_to_move_on_an_FOD-based_DEC_map_to_replace_DTI's_trademark_DEC_FA) for more details.

You can compute an FOD-based DEC map from your WM FOD image as follows:

```
fod2dec wmfod_norm.mif decfod.mif -mask dwi_mask_upsampled.mif
```

Open the resulting `decfod.mif` directly in `mrview` as a good visualisation of the WM FOD compartment from your 3-tissue CSD result. As the FOD-based DEC map is computed directly from the WM FOD image, and encodes some of its main properties visually, it is the perfect companion to the WM FOD image itself indeed. You can open both at once (to overlay the WM FODs on the FOD-based DEC map) as follows:

```
mrview decfod.mif -odf.load_sh wmfod_norm.mif
```

Within `mrview`, you can open the "*ODF display*" tool from the "*Tool*" menu/button to access more options related to the FOD visualisation. The "*scale*" setting in particular can be useful to scale the FOD size, depending on the structures you're aiming to visualise well.

### Tractography

The WM FOD resulting from 3-tissue CSD is the perfect source of information to guide fiber tractography! Different kinds of tractography algorithms as well as many different options and parameters are offered by the `tckgen` command. A simple example to run whole-brain tractography, which also shows the most important basic options, can be performed as follows:

```
tckgen wmfod_norm.mif tracks.tck -seed_image dwi_mask_upsampled.mif -select 100000 -cutoff 0.07
```

This example will run the default ("iFOD2") probabilistic tractography algorithm. The options in this example tell it to seed tracks from the entire brain mask, try to run until 100000 tracks have been successfully selected (meeting certain requirements, such as a default minimum track length, among others) and use an FOD amplitude threshold ("cutoff") of 0.07. It is very important to choose the cutoff parameter well (0.07 is only used as an example here): decreasing it will allow smaller FOD lobes to be tracked, whereas increasing it will disregard those smaller peaks, as if they were noise or otherwise unwanted features. A good cutoff value may depend on the quality of your data, and will ultimately be a trade-off between false negatives and false positives. Experiment with a few values to get a good feeling for this value for your data and experiment at hand.

You can visualise the resulting tractogram in `mrview`. This can again be overlaid on the FOD-based DEC map, as follows:

```
mrview decfod.mif -tractography.load tracks.tck -tractography.lighting true
```

Within `mrview`, you can open the "*Tractography*" tool from the "*Tool*" menu/button to access more options related to the tractogram visualisation. By default, a "slab" will be shown, centered around the currently displayed slice of the image. You can change the thickness of this slab, or even switch "*crop to slab*" off entirely, showing the whole tractogram in full 3D at once! Using other controls in `mrview`, you can e.g. rotate this to inspect it.

Take a look at the command line help page of `tckgen` for all possible options. There are also several options, such as `-include` and `-exclude` if you want to perform targetted tractography using predefined regions.

* * *

[[Go back to the documentation index]]({% link doc/index.md %})
