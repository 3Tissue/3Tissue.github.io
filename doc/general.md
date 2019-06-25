---
title: 'General usage'
description: 'A quick start guide to general command usage and a basic list of some common commands'
creatortwitter: 'MRtrix3Tissue'
---

## General usage

### Basic command names

As also documented in the [MRtrix3 paper](https://www.sciencedirect.com/science/article/abs/pii/S1053811919307281), MRtrix3 (and MRtrix3Tissue) has a large set of commands, several of which follow a common naming convention which reveals or at least hints at the purpose of those commands. Those commands' names use short or abbreviated notations of the data they operate on or output; e.g. "`mr`" for generic (MR) images, "`dwi`" for diffusion-weighted images and "`tck`" for tracks (or streamlines). Using these, there are two more common formats of command names:

* "`DataOperation`": reveals the type of data the command mainly works with and the operation the command performs, e.g. `mrconvert` converts an (MR) image to different formats, `mrinfo` provides information about an (MR) image, `tckgen` generates tracks (streamlines), `dwidenoise` performs denoising on diffusion-weighted images, etc...

* "`Data2Data`": reveals the input and output type of data for the command, where the command performs a conversion or an algorithm to generate the output based on the input, e.g. `warp2metric` computes certain metrics from spatial warps, `dwi2tensor` computes the diffusion tensor model from diffusion-weighted images, `fod2dec` computes directionally-encoded colour ("DEC") maps from fibre orientation distribution (FOD) images, etc...

Note that not *all* commands follow these conventions.

### Command line help and references

Once you've finished [installing and building]({% link doc/install/index.md %}) MRtrix3Tissue, the full list of commands is available in the `bin` folder (within the main MRtrix3Tissue folder).

When (you think) you've identified the command you need to perform something specifically, you can read its help page directly at the command line by simply typing the command name on its own (and hitting `ENTER`). You can scroll up and down the help page with the (`up` and `down`) arrow keys on your keyboard. Most commands have a synopsis and a description, which should allow you to quickly verify whether the command is indeed the right tool for the job. The help page describes which arguments the command requires, and which options are available.

Finally, the help page also lists the scientific reference(s) for the method(s) offered by a command. These references are located all the way at the bottom of the help page (you can scroll down faster using the `Page Down` key or directly jump to the bottom using the `End` key on your keyboard).

* * *

[[Go back to the documentation index]]({% link doc/index.md %})
