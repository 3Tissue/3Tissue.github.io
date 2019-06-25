---
title: 'Windows installation'
description: 'Additional information for installation on Windows'
creatortwitter: 'MRtrix3Tissue'
---

## Windows installation

This page provides information for installing MRtrix3Tissue on Windows systems. This information is provided in addition to the [general installation instructions]({% link doc/install/index.md %}).

Note that, while it's technically possible to run MRtrix3Tissue on Windows, it's not the most convenient or enjoyable experience. Since diffusion MRI preprocessing typically relies on [FSL](https://fsl.fmrib.ox.ac.uk) and/or [ANTs](http://stnava.github.io/ANTs), a Unix-based system is even more encouraged instead. You might want to consider a virtual machine or similar if you're stuck on a Windows system.

### Requirements

These are the basic requirements to install and run all command line tools:

* C++11 compliant compiler
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

All these dependencies are installed below by the MSYS2 package manager.

### Installing dependencies

1. Download and install the most recent 64-bit MSYS2 installer from [http://msys2.github.io](http://msys2.github.io) (msys2-x86_64-*.exe), and follow the installation instructions from the [MSYS2 wiki](https://github.com/msys2/msys2/wiki/MSYS2-installation).

2. Run the program "**MinGW-w64 Win64 Shell**" from the start menu.

3. Update the system packages, by following [the instructions](https://github.com/msys2/msys2/wiki/MSYS2-installation#iii-updating-packages):

   ```
   pacman -Syuu
   ```

   Close the terminal, start a new "**MinGW-w64 Win64 Shell**", and repeat as necessary until no further packages are updated.

   It was reported that this MSYS2 system update gives a number of instructions, including: terminating the terminal when the update is completed, and modifying the shortcuts for executing the shell(s). Although these instructions are not as prominent as they could be, it is absolutely *vital* that they are followed *strictly correctly*!

4. From the "**MinGW-w64 Win64 Shell**" run:

   ```
   pacman -S git python pkg-config mingw-w64-x86_64-gcc mingw-w64-x86_64-eigen3 mingw-w64-x86_64-qt5 mingw-w64-x86_64-fftw mingw-w64-x86_64-libtiff mingw-w64-x86_64-libpng
   ```

   Sometimes `pacman` may fail to find a particular package from any of the available mirrors. If this occurs, you can download the relevant package from [SourceForge](https://sourceforge.net/projects/msys2/files/REPOS/MINGW/x86_64/): place both the package file and corresponding .sig file into the `/var/cache/pacman/pkg` directory, and repeat the `pacman` call above.

   Sometimes `pacman` may refuse to install a particular package, claiming e.g.:

   ```
   error: failed to commit transaction (conflicting files)
   mingw-w64-x86_64-eigen3: /mingw64 exists in filesystem
   Errors occurred, no packages were upgraded.
   ```

   Firstly, if the offending existing target is something trivial that can be deleted, this is all that should be required. Otherwise, it is possible that MSYS2 may mistake a *file* existing on the filesystem as a pre-existing *directory*; a good example is that quoted above, where `pacman` claims that directory `/mingw64` exists, but it is in fact the two files `/mingw64.exe` and `/mingw64.ini` that cause the issue. Temporarily renaming these two files, then changing their names back after `pacman` has completed the installation, should solve the problem.

### Installing MRtrix3Tissue

Once the required dependencies are present and installed, you can refer to the [general installation instructions]({% link doc/install/index.md %}) to install MRtrix3Tissue.

### Problems

Problems typically call for [feedback or support]({% link feedback-support.md %}).


* * *

[[Go back to the documentation index]]({% link doc/index.md %})
