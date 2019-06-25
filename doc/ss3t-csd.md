---
title: 'Single-Shell 3-Tissue CSD (SS3T-CSD)'
description: 'Information on how to use the SS3T-CSD algorithm'
creatortwitter: 'MRtrix3Tissue'
---

## Single-Shell 3-Tissue CSD (SS3T-CSD)

**Single-Shell 3-Tissue CSD (SS3T-CSD)** is a method that allows you to obtain *3-tissue CSD results* from just *single-shell (+b=0)* diffusion MRI data. This is otherwise *not* possible using the Multi-Shell Multi-Tissue CSD (MSMT-CSD) method, which is limited to resolve only 2 tissue compartments from such data. The core mechanism behind SS3T-CSD is explained [here](https://www.researchgate.net/publication/301766619_A_novel_iterative_approach_to_reap_the_benefits_of_multi-tissue_CSD_from_just_single-shell_b0_diffusion_MRI_data). Note the initial name (in the linked abstract) was slightly different; it was changed very shortly after, since initial feedback made us realise the original name was too confusing.

The SS3T-CSD method is now publicly available in MRtrix3Tissue (from v5.1.0 onwards), via the `ss3t_csd_beta1` command. We're looking to get ongoing feedback from your experience(s) using it on your data; the method is now pretty robust and the default behaviour is optimised for typical use cases. Note though, that for particular kinds of data or data quality, a certain parameter can be changed (from its default value) to get the best performance. See below for more information.

The steps below assume you're using MRtrix3Tissue v5.1.0 or later, and your data is already preprocessed (refer to the [basic single-subject pipeline]({% link doc/single-subject.md %})). The example below will refer to the preprocessed data as `dwi-prep.mif` and a corresponding (brain) mask `mask.mif` for brevity.

### 3-tissue response function estimation

It's important to not confuse the SS3T-CSD method with the 3-tissue *response function estimation* step. Even though SS3T-CSD relies on good 3-tissue response functions, estimating those response functions is a *separate* step, preceding SS3T-CSD.

3-tissue response functions can be estimated directly from the data themselves using the `dwi2response dhollander` command. In MRtrix3Tissue, this runs the algorithm with the latest improvements to single-fibre WM response function estimation [as introduced at ISMRM 2019](https://www.researchgate.net/publication/331165168_Improved_white_matter_response_function_estimation_for_3-tissue_constrained_spherical_deconvolution). A video of the ISMRM 2019 talk is also available [over here](https://youtu.be/7yPSFgLt8CA).

You can run 3-tissue response function estimation as follows:

```
dwi2response dhollander dwi-prep.mif response_wm.txt response_gm.txt response_csf.txt
```

You can also add the `-voxels response_voxels.mif` option to this command. The `response_voxels.mif` image can then be opened in `mrview` (and overlaid on the diffusion MRI data for spatial reference). This allows to directly inspect which voxels the algorithm ended up sourcing the 3-tissue response functions from.


### SS3T-CSD

Once good 3-tissue response functions have been obtained from the data, you can run SS3T-CSD as follows:

```
ss3t_csd_beta1 dwi-prep.mif response_wm.txt wmfod.mif response_gm.txt gm.mif response_csf.txt csf.mif -mask mask.mif
```

The ordering of inputs/outputs is *strict* (WM - GM - CSF, in *that* order).

The SS3T-CSD method is by default optimised for (and generally performs best on) high b-value, high angular resolution data; e.g. a b-value of 2500 s/mm&sup2; or higher and 45 gradient directions or more. Since one of the essential keys to getting 3-tissue compartments from the data lies in using the b=0 data as well, it's also good to have a few b=0 images; or to put it diffently: if you only have *one* b=0 image, a lot depends directly on that *one* image, so should there be any problems with that one b=0 image, this could be directly reflected in the output.

However, these recommendations aren't strict at all: we've been able to run SS3T-CSD relatively well on low b-value (1000 s/mm&sup2; or lower) and low angular resolution (30 gradient directions or less) diffusion MRI data with only a single b=0 image. The results won't be as good then as in optimal conditions, but there should still be an important improvement over single-tissue CSD, if the data quality allows for it.

### Parameters

Generally, SS3T-CSD starts out regarding all signal as "GM-like" and during the SS3T-CSD internal iterations, it trades parts of the "GM-like" compartment in for "WM-like" and "CSF-like" compartments, depending on evidence of those in the diffusion MRI signal itself. It does this "trading" at a certain pace, over iterations. Refer to [the original explanation](https://www.researchgate.net/publication/301766619_A_novel_iterative_approach_to_reap_the_benefits_of_multi-tissue_CSD_from_just_single-shell_b0_diffusion_MRI_data) for more insight into the mechanism by which the iterations trade in GM-like for WM-like/CSF-like compartments.

So this then naturally gives rise to 2 possible (kinds of) parameters: the *number of iterations* as well as a parameter that controls this *pace*. However, for most data of reasonable quality (see above), we recommend to *not* change these parameters from their default values.

By default, SS3T-CSD will perform an initialisation, followed by 3 iterations (each taking about as long as a single normal run of the MSMT-CSD algorithm). It's generally not recommended to lower the number of iterations. Should you increase it, more of the GM-like compartment might be traded in for the WM-like and CSF-like compartments. For ongoing iterations, this process slows down though (as the evidence for additional WM-like and CSF-like signal gradually becomes less and less). We recommend to stick to the default number of (3) iterations, and use the other parameter which controls the *pace* instead if (and only if) there is a need to, due to particular data qualities or limitations.

The parameter or property which (indirectly) controls the pace at which each iteration advances is the weight of the b=0 images (as a whole) in the fitting process that happens at each iteration. This parameter is offered as a percentage, i.e., the b=0 image contribution is weighted as a percentage of the number of (non b=0) diffusion weighted images (refer to the command line documentation for the option). By default, this is set to 10 (per cent), i.e., as if there were 10 per cent b=0 images in the data, relative to (non b=0) diffusion weighted images. Having this parameter as such a percentage (set to a default value of 10 per cent) means the algorithm can produce similar results, regardless of the actual *number* of b=0 images (relative to (non b=0) diffusion weighted images) in the data. Again, we recommend to stick with the default. For typical human data (and 3-tissue response functions), *increasing* this percentage *lowers* the pace of the algorithm over iterations. Sometimes, for lower b-value data, or more generally *lower contrast-to-noise ratio* data (both in the angular as well as the radial domain), the default pace of the algorithm ends up being inherently a bit faster: more GM-like signal is traded in at each iteration, so you end up with a smaller GM-like compartment (and less GM-like signal removal from the WM FOD) in the end. In such cases, it might be worthwhile experimenting with increasing the b=0 contribution percentage. However, essentially the limitation of such data *remains* its low contrast(-to-noise ratio), in the angular and/or radial domain. Ultimately, these contrasts are the features which allow SS3T-CSD to tease the tissues apart. In a way, it's not a bad thing the algorithm naturally errs on the side of more WM-like signal when contrast-to-noise properties provide limited information: as there is less evidence in the data to separate tissues well, you wouldn't want to lose possible parts of the WM-like (FOD!) compartment. You already gain a better WM FOD (compared to single-tissue or 2-tissue CSD) by removing some GM-like signal in any case; but you wouldn't want to start *over*-estimating GM-like signal to the detriment of the WM FOD!

So in conclusion: we still *highly* recommend to stick with the defaults, "by default". Proceed with great care otherwise, and start with gently changing the b=0 contribution percentage in such exceptional cases (and closely inspect the impact on the result). If your data has certain limitations, well, acknowledge and respect those limitations. In any case, please consider providing feedback, so we can learn and improve if possible!

### Description and citation

If you use `dwi2response dhollander` and SS3T-CSD in your work, you can describe this (including citations) using the following or a similar formulation:

*"Response functions for single-fibre WM as well as GM and CSF were estimated from the data themselves using an unsupervised method ([Dhollander et al 2019](https://www.researchgate.net/publication/331165168_Improved_white_matter_response_function_estimation_for_3-tissue_constrained_spherical_deconvolution)). Single-Shell 3-Tissue CSD (SS3T-CSD) was performed to obtain WM-like FODs as well as GM-like and CSF-like compartments in all voxels ([Dhollander & Connelly, 2016](https://www.researchgate.net/publication/301766619_A_novel_iterative_approach_to_reap_the_benefits_of_multi-tissue_CSD_from_just_single-shell_b0_diffusion_MRI_data)), using MRtrix3Tissue ([https://3Tissue.github.io](https://3Tissue.github.io)), a fork of MRtrix3 ([Tournier et al. 2019](https://www.sciencedirect.com/science/article/abs/pii/S1053811919307281))."*

Of course, make sure you've introduced the appropriate abbreviations (e.g. "FOD") either earlier in your text, or adapt accordingly. See also the "[About MRtrix3Tissue]({% link about.md %})" page for general MRtrix3Tissue (citation) information.

### Provide feedback!

We hope to learn from your experience(s) with the SS3T-CSD method. Please consider [providing feedback]({% link feedback-support.md %}), for either good or bad results or to ask questions in case you're unsure about *anything at all*. Make sure to provide some basic details about your data quality (b-value, number of gradient directions and number of b=0 images) and to include screenshots of your final WM, GM and CSF compartment images, and of your WM FODs. If you're unsure about other steps, e.g. whether your response functions are sensible, etc... feel free to include as much information and details as you can. Your feedback allows us to improve this information!

* * *

[[Go back to the documentation index]]({% link doc/index.md %})
