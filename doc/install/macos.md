---
title: 'macOS installation'
description: 'Additional information for installation on macOS'
creatortwitter: 'MRtrix3Tissue'
---

## macOS installation

This page provides information for installing MRtrix3Tissue on macOS systems. This information is provided in addition to the [general installation instructions]({% link doc/install/index.md %}).

### Requirements

These are the basic requirements to install and run all command line tools:

* C++11 compliant compiler (e.g. clang in Xcode)
* Python version >= 2.7 (already included in macOS)
* zlib compression library (already included in macOS)
* Eigen version >= 3.3

These are additional requirements to install and run the GUI (`mrview` and `shview`):

* Qt version >= 5.1
* an OpenGL 3.3 compliant graphics card and corresponding software driver (OpenGL 3.3 is supported across macOS versions >= 10.9)

These are optional dependencies:

* libTIFF version >= 4.0 (for TIFF support)
* FFTW version >= 3.0 (for improved performance in certain applications)
* libpng (for PNG support)

### Installing dependencies

1. Update macOS to version 10.10 (Yosemite) or higher (OpenGL 3.3 will typically not work on older versions).

2. Install XCode from the Apple Store.

3. Install Eigen3 and Qt5.

   The most convenient way is to use either the Homebrew or MacPorts package manager. Don't mix either of these though, as this typically causes a lot of issues that then become very hard to revert. So pick one, and stick with it.

   * Homebrew:
     * Install Eigen3: `brew install eigen`
     * Install Qt5: `brew install qt5`
     * Install pkg-config: `brew install pkg-config`
     * Add Qt's binaries to your path: `export PATH='brew --prefix'/opt/qt5/bin:$PATH`

   * MacPorts:
     * Install Eigen3: `port install eigen3`
     * Install Qt5: `port install qt5`
     * Install pkg-config: `port install pkgconfig`
     * Add Qt's binaries to your path: `export PATH=/opt/local/libexec/qt5/bin:$PATH`

4. Install the TIFF, FFTW and PNG libraries.

   As before, pick one and stick with it.

   * Homebrew:
     * Install TIFF: `brew install libtiff`
     * Install FFTW: `brew install fftw`
     * Install PNG: `brew install libpng`

   * MacPorts:
     * Install TIFF: `port install tiff`
     * Install FFTW: `port install fftw-3`
     * Install PNG: `port install libpng`

### Installing MRtrix3Tissue

Once the required dependencies are present and installed, you can refer to the [general installation instructions]({% link doc/install/index.md %}) to install MRtrix3Tissue.

### Problems

Problems typically call for [feedback or support]({% link feedback-support.md %}).


* * *

[[Go back to the documentation index]]({% link doc/index.md %})
