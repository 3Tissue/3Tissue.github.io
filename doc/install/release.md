---
title: 'Release information'
description: 'Information about MRtrix3Tissue releases'
creatortwitter: 'MRtrix3Tissue'
---

## Release information

### Latest release

The latest MRtrix3Tissue releases can be found [here](https://github.com/3Tissue/MRtrix3Tissue/releases), as well as all release notes and history. You can download the code for each release, or install (and update) MRtrix3Tissue directly via `git`. Refer to the [general installation instructions]({% link doc/install/index.md %}) for details on the latter.

If you want to be automatically notified of new releases, you can _"watch"_ the [repository](https://github.com/3Tissue/MRtrix3Tissue) by clicking the _"watch"_ button at the top. You can also follow the [MRtrix3Tissue Twitter account](https://twitter.com/MRtrix3Tissue), where releases will be announced too.

### Version numbering

MRtrix3Tissue is a fork of MRtrix3. This means it includes most MRtrix3 functionality out of the box, but also that version information may get confusing if not managed well. MRtrix3Tissue version numbering is separate from MRtrix3. An effort has been made to make this as overtly clear as possible for users, in a couple of ways:

* The so-called version _"tags"_ for MRtrix3Tissue releases always follow the same format: `3Tissue_vx.y.z`, e.g. `3Tissue_v5.1.0`.
* The _"major"_ version number (the first number after `3Tissue_v`) has skipped version "3", so similar version numbers can be avoided.

### Changes since MRtrix3 "RC3"

MRtrix3Tissue started its life with version v0.0.0 (`3Tissue_v0.0.0`), which is *identical* to MRtrix3 Release Candidate 3 (`3.0_RC3`), but has initially quickly been updated with MRtrix3's "development" branch during June-July-August 2019. Since these upstream MRtrix3 developments have stretched out over more than a year (starting from May 2018), many things have been updated but also *changed*.

For users who have used MRtrix3 Release Candidate 3 (`3.0_RC3`) before, here's a short list of the most critical name changes of major commands, assuming you're using at least MRtrix3Tissue v5.1.0 (`3Tissue_v5.1.0`):

* `dwipreproc` ==> `dwifslpreproc`
* `average_response` ==> `responsemean`
* `mrresize` and `mrpad` and `mrcrop` ==> `mrgrid`
* `dwinormalise` and `dwiintensitynorm` ==> `dwinormalise` (*only* for old single-tissue analyses; `mtnormalise` is used instead for new 3-tissue analyses)

The interface (arguments, options) and/or default behaviour of some of the above have also changed. Please refer to their command line documentation for details. Here are some other commands for which the interface or default behaviour has changed (please note that this list is *not* exhaustive):

* `foreach` shows progress differently and has some new options; basic usage remains the same (i.e. backward compatible)
* `dwi2tensor` changed default fitting strategy from ordinary least-squares (OLS) followed by iterative weighted least-squares (IWLS), to *weighted least-squares (WLS)* followed by IWLS
* `dwidenoise` changed its default behaviour to a new improved noise estimator (see command line documentation for reference), has new edge voxel handling and a new default patch size based on the number of volumes in the data
* `mrtransform` now requires (mandatory) to specify whether to perform FOD reorientation (`-reorient_fod yes`) or not (`-reorient_fod no`)
* `mrtheshold` has several changes and improvements, including a new basic user interface and option; although most existing basic user interface options remain available
* `dwibiascorrect` now has an algorithm selection like some other commands; to see the help page of a specific algorithm, type e.g. `dwibiascorrect ants`

Another important change is that several commands that output text files, will now add comments (lines starting with `#`, followed by other text) in those text files to describe certain _"header"_ information. For example, `dwi2response dhollander` will add 2 lines of comments at the start of each output response function text file: these document the full command that was run (by the user) to generate those outputs (and the version of the software), as well as the b-values of the response function. This change is particularly important to be aware of for users that have their own scripts or processing pipelines that rely on reading these text files or extracting information from them (potentially in external data processing software packages).

Finally, MRtrix3Tissue currently brings a few specific improvements:

* `dwi2response dhollander` now uses the improved 2019 version for single-fibre WM response function estimation, allowing for better subsequent 2-tissue and 3-tissue CSD fits, for both multi-shell as well as single-shell (+b=0) data. The old algorithm is deprecated, but still temporarily available as `dwi2response dholl_old` for reference. See this ISMRM 2019 [abstract](https://www.researchgate.net/publication/331165168_Improved_white_matter_response_function_estimation_for_3-tissue_constrained_spherical_deconvolution) and [talk](https://youtu.be/7yPSFgLt8CA) for details.
* **Single-Shell 3-Tissue CSD (SS3T-CSD)** has been included (admittedly, *finally!*) from MRtrix3Tissue v5.1.0 (`3Tissue_v5.1.0`) onwards, currently with a "beta" label. It's been thoroughly tested with private beta test users though, and should now be pretty solid for a decent range of data and cases. Read [this page]({% link doc/ss3t-csd.md %}) for essential information. I'd also love to receive your [feedback]({% link feedback-support.md %}).

If you want to get started immediately, take a look at the [basic single-subject pipeline]({% link doc/single-subject.md %}), which also serves as an excellent tutorial.


* * *

[[Go back to the documentation index]]({% link doc/index.md %})
