---
title: 'Linux installation'
description: 'Additional information for installation on Linux'
creatortwitter: 'MRtrix3Tissue'
---

## Linux installation

This page provides information for installing MRtrix3Tissue on Linux systems. This information is provided in addition to the [general installation instructions]({% link doc/install/index.md %}).

### Requirements

These are the basic requirements to install and run all command line tools:

* C++11 compliant compiler (GCC version >= 4.9, clang)
* Python version >= 2.7
* zlib compression library
* Eigen version >= 3.3

These are additional requirements to install and run the GUI (`mrview` and `shview`):

* Qt version >= 4.8
* an OpenGL 3.3 compliant graphics card and corresponding software driver

These are optional dependencies:

* libTIFF version >= 4.0 (for TIFF support)
* FFTW version >= 3.0 (for improved performance in certain applications)
* libpng (for PNG support)

### Installing dependencies

The procedure depends on your system, and might change over time (with newer versions of operating systems and packages). The commands below are suggestions that have worked successfully for people (at least at some point).

Ubuntu and derivatives (e.g. Linux Mint):

```
sudo apt-get install git g++ python libeigen3-dev zlib1g-dev libqt4-opengl-dev libgl1-mesa-dev libfftw3-dev libtiff5-dev libpng-dev
```

RPM-based distributions (e.g. Fedora, CentOS, SUSE Linux):

```
sudo yum install git g++ python eigen3-devel zlib-devel libqt4-devel libgl1-mesa-dev fftw-devel libtiff-devel libpng-devel
```

On Fedora 24, this was reported to work:

```
sudo yum install git gcc-c++ python eigen3-devel zlib-devel qt-devel mesa-libGL-devel fftw-devel libtiff-devel libpng-devel
```

Arch Linux:

```
sudo pacman -Syu git python gcc zlib eigen qt5-svg fftw libtiff libpng
```

### Installing MRtrix3Tissue

Once the required dependencies are present and installed, you can refer to the [general installation instructions]({% link doc/install/index.md %}) to install MRtrix3Tissue.

### Problems

Problems typically call for [feedback or support]({% link feedback-support.md %}).


* * *

[[Go back to the documentation index]]({% link doc/index.md %})
