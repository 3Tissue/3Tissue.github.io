---
title: 'Install and update'
description: 'General instructions for quick installation and update'
creatortwitter: 'MRtrix3Tissue'
---

## Installation

Below are some general quick installation instructions, mostly aimed at Unix-based systems (e.g. Linux or Apple macOS). While it's technically possible to run MRtrix3Tissue on Windows, it's not the most convenient or enjoyable experience. Since diffusion MRI preprocessing typically relies on [FSL](https://fsl.fmrib.ox.ac.uk) and/or [ANTs](http://stnava.github.io/ANTs), a Unix-based system is even more encouraged.

In addition to the below information, further OS-specific instructions are also available: [Linux]({% link doc/install/linux.md %}), [macOS]({% link doc/install/macos.md %}), [Windows]({% link doc/install/windows.md %}).

If you've used an _older_ version of MRtrix3 before, such as the "release candidate 3 (RC3)", you might want to read some [release information]({% link doc/install/release.md %}) first to inform yourself of the most critical changes in usage (e.g. name changes of certain commands).

### Quick install

1. Install dependencies; these include: Python (>=2.7), a C++ compiler with full C++11 support (`g++` 4.9 or later, `clang++`), Eigen (>=3.3), zlib, OpenGL (>=3.3) and Qt (>=4.8 or at least 5.1 on macOS). Refer to the OS-specific instructions for details: [Linux]({% link doc/install/linux.md %}), [macOS]({% link doc/install/macos.md %}), [Windows]({% link doc/install/windows.md %}).

2. Clone Git repository, configure and build:

   ```
   git clone https://github.com/3Tissue/MRtrix3Tissue.git MRtrix3Tissue
   
   cd MRtrix3Tissue
   
   ./configure
   
   ./build
   ```

3. Set the `PATH` (assuming you're using the Bash shell):

   Run the provided `set_path` script:

   ```
   ./set_path
   ```

   This will automatically update the `~/.bashrc` startup file (or `~/.bash_profile` on macOS) for you. You can also instruct the `set_path` script to explicitly update a specific startup file, e.g.:

   ```
   ./set_path ~/.profile
   ```

   You can also edit the appropriate startup file for your system manually and add this line _at the end_:

   ```
   export PATH=/<edit as appropriate>/MRtrix3Tissue/bin:$PATH
   ```

   Once you've run the `set_path` script or edited a startup file, you have to close and re-open a terminal for this startup file to (automatically) run and your `PATH` to be correctly set.

4. Test your installation:

    Command line:

    ```
    mrinfo
    ```

    GUI:

    ```
    mrview
    ```

### Keep MRtrix3Tissue up to date

1. Update your installation by opening a terminal in the MRtrix3Tissue folder, and type:

   ```
   git pull

   ./build
   ```

2. If `./build` doesn't work immediately, you may need to re-run the `configure` script:

   ```
   ./configure
   ```

   and re-run `./build` again. You don't have to update or set your `PATH` again if it was also set before.

### Build a specific version of MRtrix3Tissue

You can build a _specific_ version of MRtrix3Tissue by checking out its so-called _"tag"_, and using the same procedure as above to build it:

```
git checkout 3Tissue_v5.1.0

./configure

./build
```

### Optimising performance

By default, the `configure` script's output instructs the `build` script to produce generic code suitable for any current CPU. However, you can also have this process produce code that's more specifically tailored to the specific CPU in your computer. As this can improve performance of your MRtrix3Tissue installation, it's worth considering if your installation will only run on your computer (and not be moved to another computer).

To achieve this, you need to set the `ARCH` environment variable to `native` right before running `./configure`:

```
export ARCH=native

./configure

./build
```

### Not installing the GUI

If you're installing MRtrix3Tissue on a system where you don't plan to use `mrview` and `shview` (e.g. a server, HPC system or cluster), you can avoid building these GUI tools. This also means Qt and OpenGL won't be required.

To achieve this, you need to provide the `-nogui` option to `./configure`:

```
./configure -nogui

./build
```

### Problems

Problems typically call for [feedback or support]({% link feedback-support.md %}).

* * *

[[Go back to the documentation index]]({% link doc/index.md %})
