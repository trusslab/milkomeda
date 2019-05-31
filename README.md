# Build Milkomeda from Source

Copyright (c) 2018 University of California, Irvine. All rights reserved.

See our [Milkomeda paper](https://dl.acm.org/citation.cfm?id=3243772) for technical details.
Authors: Zhihao Yao, Saeed Mirzamohammadi, Ardalan Amiri Sani, Mathias Payer

This document is shared under the GNU Free Documentation License WITHOUT ANY WARRANTY. See <https://www.gnu.org/licenses/> for details. The build instructions are based on [Google's guide to building Chromium on Linux](https://chromium.googlesource.com/chromium/src/+/lkcr/docs/linux_build_instructions.md) and [The LineageOS Project's guide to build for bullhead](https://wiki.lineageos.org/devices/bullhead/build).

_______________________________

## System Requirements

* At least 200 GB of free disk space, and another 100 GB if you wish to build Chromium WebGL security checks from source.
* A relatively recent 64-bit Intel computer, preferably 16GB+ RAM.
* Ubuntu 16.04 LTS. Other distributions are mostly unsupported by Google's build tools.
* A Nexus 5X smartphone.

It is recommended to backup your system before proceeding. 

## Install Build Tools

To install build dependencies, Android platform tools, and `repo`, run:

```bash
sudo apt-get install openjdk-8-jdk
sudo apt-get install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev libesd0-dev liblz4-tool libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```

Install `repo` and Android platform tools by running:

```bash
mkdir ~/bin; cd ~/bin
wget https://dl.google.com/android/repository/platform-tools-latest-linux.zip
unzip platform-tools-latest-linux.zip -d ~/bin/
rm platform-tools-latest-linux.zip
echo "# set Android SDK platform tools path" >> ~/.profile
echo "if [ -d \"\$HOME/bin/platform-tools\" ] ; then" >> ~/.profile
echo "    PATH=\"\$HOME/bin/platform-tools:\$PATH\"" >> ~/.profile
echo "fi" >> ~/.profile
source ~/.profile

curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

## Get the code

You will need to create a Milkomeda home folder (e.g. `~/bullhead_milkomeda`), and `cd` to the folder. **The rest of this section assumes you are in the folder.** Run:

```bash
mkdir ~/bullhead_milkomeda
cd ~/bullhead_milkomeda
repo init -u https://github.com/LineageOS/android.git -b cm-14.1
```

Fetch the code by running:

```sh
repo sync
```

This will take a while. When it is done, run:

```
source build/envsetup.sh
breakfast bullhead
```

Now you are ready to pull Milkomeda repositories. First, we need to get additional build tools, scripts, and prebuilt binaries by running:

```bash
# Make sure you are in Milkomeda home folder
git clone https://github.com/trusslab/milkomeda_tools_binaries.git
mv ./milkomeda_tools_binaries/* ./
rm -rf ./milkomeda_tools_binaries
```

Next, pull Milkomeda repositories by typing:

```bash
cd external/jemalloc/
git remote add origin https://github.com/trusslab/milkomeda_bullhead_jemalloc.git
git checkout 836d536
git pull origin release
git checkout 836d536
cd -

cd frameworks/native/
git remote add origin https://github.com/trusslab/milkomeda_bullhead_native.git
git checkout c2e8ee8
git pull origin release
# Git will prompt a merge message. Press Ctrl-C to exit.
git checkout c2e8ee8
cd -

cd frameworks/base/
git remote add origin https://github.com/trusslab/milkomeda_bullhead_base.git
git checkout 7467536
git pull origin release
git checkout 7467536
cd -

cd kernel/lge/bullhead/
git remote add origin https://github.com/trusslab/milkomeda_bullhead_kernel.git
git checkout cdc93dc
git pull origin release
git checkout cdc93dc
cd -

cd bionic/
git remote add origin https://github.com/trusslab/milkomeda_bullhead_bionic.git
git checkout d207f78
git pull origin release
git checkout d207f78
cd -

# Our development is based on the cm-14.1 branch (2017-08). 
# Some repositories receive updates since then. 
# We need to checkout them back to an older version to avoid compiler errors.

cd external/skia
git checkout da4a326
cd -

cd packages/apps/CMParts
git checkout 8ba6fe3
cd -

cd packages/providers/MediaProvider
git checkout 03abed5
cd -

cd packages/providers/DownloadProvider
git checkout 581eab3
cd -

cd packages/apps/Settings
git checkout 2701395
cd -

cd packages/apps/PackageInstaller
git checkout 1b36092
cd -
```

## Build System Image

This section builds a clean system image. **You must completely build it at least once before building Milkomeda programs**, otherwise the compiler will complain about missing files later. 

To start the build, run:

```bash
source build/envsetup.sh
breakfast bullhead
croot
brunch bullhead
```

Note: **Do not flush your Nexus 5X with this build unless you have retrieved device-specific blobs**. Please see [The LineageOS Project's guide to build for bullhead](https://wiki.lineageos.org/devices/bullhead/build) for a complete guide.

Note: If Jack (Java compiler) runs out of memory, run this command and reboot. 

```bash
echo "export ANDROID_JACK_VM_ARGS=\"-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4G\"" >> ~/.bashrc
```

Note: If compiling fails due to missing binaries in `out/host/linux_x86/bin/`, run this command and retry:

```bash
mv host/* out/host/linux-x86/bin/
```

## Build Milkomeda Kernel

Note: Kernel must be built with unmodified `bionic/` and `external/jemalloc` repositories. If you want to build kernel later, make sure these repository HEADs are pointed to the correct commits:

```bash
cd bionic
git checkout d207f78
cd -

cd external/jemalloc
git checkout 836d536
cd -
```

To start the build, run:

```bash
cd kernel/lge/bullhead/
git checkout release
cd -

source build-kernel.sh
```

Note: to run Milkomeda, you must disable SELinux (or set it to `permissive` mode ) temporarily. We have already set it to `permissive` mode in the kernel. See changes in  `security/selinux/selinuxfs.c`.

## Build Milkomeda User Space Programs

First, we need to switch the following repositories to our Milkomeda branch. Make sure you are in Milkomeda root folder. Run:

```bash
cd external/jemalloc/
git checkout release
cd -

cd frameworks/native/
git checkout release
cd -

cd frameworks/base/
git checkout release
cd -

cd bionic/
git checkout release
cd -

source build-milkomeda-all.sh
```

## Build Security Checks (Optional)

We have already populated `<Milkomeda home>/chr_libs` with prebuilt binaries. If you prefer to use the prebuilt binaries, simply skip this section. 

This section provides guides to build the Chromium WebGL security checks into dynamic libraries. Once the build finishes, you will need to copy the libraries to `<Milkomeda home>/chr_libs/`.

### Get `depot_tools`

Even if you already have Chromium `depot_tools`, you still need to clone our repository. Get `milkomeda_depot_tools` by running:

```bash
git clone https://github.com/trusslab/milkomeda_depot_tools.git
cd milkomeda_depot_tools
git apply prevent_update.diff
cd ..
```

You might want to add the path to `depot_tools` to your `~/.bashrc`:

```bash
export PATH="$PATH:/path/to/milkomeda_depot_tools"
```

Note: **do not** use `~` on PATH, otherwise the later commands will fail. Rather, you should use either `$HOME` or the absolute path:

```bash
export PATH="$PATH:${HOME}/milkomeda_depot_tools"
```

Note: If you have previously installed Chromium `depot_tools`, you might want to update your `~/.bashrc` with the new path.

### Fetch Milkomeda Chromium

Create a `chromium` directory and `cd` to it (you can call this whatever you like and put it wherever you like, as long as the full path has no spaces. Below we assume the path is `~/chromium`):

```sh
mkdir ~/chromium && cd ~/chromium
```

Run the following command to fetch the code and its dependencies:

```sh
fetch --nohooks chromium
```

Type `n` when it prompts `OK to update it to https://chromium.googlesource.com/chromium/tools/depot_tools.git ? [Y/n]`.

Note: `fetch` will clone the repository at `https://github.com/trusslab/milkomeda_chromium.git` into `src`. This command might take a while depending on your Internet connection. Expect a few hours.

The remaining instructions assume you are in `~/chromium/src`:

```bash
cd src/
```

### Install Android build dependencies

To install the build tools, run:
```bash
# type 'yes' if the script asks for installing dev software
build/install-build-deps-android.sh
```

These command will download additional binaries and dependencies:
```bash
echo "target_os = [ 'android' ]" >> ../.gclient
gclient sync
```

### Setup the build

To set up a new build, run:

```sh
gn gen out/decoder
```

Note: this will create `~/chromium/out/decoder`. You can rename `decoder` whatever you want, just make sure it is under `out/`.

Then run:

```sh
gn args out/decoder
```

The command will automatically open a config file with `vim`. Append the following lines to the file:

```sh
target_os = "android"
target_cpu = "arm64"
is_debug = false
dcheck_always_on = false
is_component_build = true
```

### Build WebGL security checks
To build, run:

```sh
ninja -C out/decoder/ gpu/command_buffer/service:service
```

Note: to speed up your build, you might want to add `-j` flag to enable multi-thread. For example,

```sh
ninja -j16 -C out/decoder/ gpu/command_buffer/service:service
```

Once the build finishes, copy `out/decoder/*.so` to `<Milkomeda home>/chr_libs/`.

## Install Milkomeda

### Install system image

Please install a clean `LineageOS cm-14.1` image. You may want to use our prebuilt image [lineage-14.1-20180515-UNOFFICIAL-bullhead.zip](https://github.com/trusslab/milkomeda_tools_binaries/releases/download/v1.0/lineage-14.1-20180515-UNOFFICIAL-bullhead.zip), or you can build your own (see [Build System Image](#build-system-image)). 

Install instructions are available [here](https://wiki.lineageos.org/devices/bullhead/install). Please follow the guide to unlocking the bootloader, Install a custom recovery, and install the `.zip` file using `adb sideload`. Please wipe the phone completely before the installation.

After a successful installation, enable `Developer Options` by clicking `About Phone` -> `Build Number` seven times, then enable `Android Debug` and `ADB root access` in `Developer Options`.

### Install Milkomeda

To install Milkomeda, run:

```bash
# Make sure you are in Milkomeda root folder
source setup_blobs.sh
source install-milkomeda-all.sh
source install-kernel.sh
```

Note: If you rebuild kernel, please make sure to `git checkout` back the Milkomeda repositories, and rebuilt them with `build-milkomeda-all.sh`. `install-milkomeda-all.sh` assumes all Milkomeda binaries are up-to-date.

Note: `lib65` contains binaries that we manually modified with `sed`. For enhanced security, the graphics libraries and most of their dependencies will use libt (a second libc loaded in the secure domain) instead of the insecure libc. A list of replaced libraries is available [here](library_list.md).

Note: every time the `framework.jar` is modified, the next boot will take longer. Expect a few minutes.

### Install test cases

Next, build and install OpenGL applications with [`Android Studio`](https://developer.android.com/studio/). Download the applications code by cloning:

```bash
https://github.com/trusslab/Learn-OpenGLES-Tutorials.git
https://github.com/trusslab/android-ndk.git
```

Note: We use Android Studio version 3.1.3, SDK API level 25, and NDK version r14b. Other versions should also work, but we haven't tested.

Note: We included a GL3JNI library path in `<Milkomeda_home_path>/frameworks/base/cmds/app_process_milkomeda/shield.cpp`. Please **make sure you have installed GL3JNI** before running Milkomeda with any other applications. To make sure we have the correct application path, **please manually uninstall an application before installing it again**. Alternatively, you can remove the GL3JNI library path from `shield.cpp` and still run other test cases.

## Run Milkomeda

Switch to your Milkomeda home folder. Open a new terminal (to avoid using a different `adb` after a build) and run:

 ```bash
adb root; adb logcat
 ```

Now, **close the phone's display by pressing the side button**. Open another terminal, launch a test case by running the corresponding script. You can `cat` the `launch*.sh` scripts to see which application a script will launch. **Once the script has finished its execution, open the phone's display and swipe up to unlock the screen.** 

```bash
source launch.sh # or launch2.sh
```

For the Learn-OpenGLES-Tutorials test cases (`launch_lesson5.sh` and `launch_lesson7.sh`), **wait for around 3 seconds after the launch script has finished its execution**, and then open the phone's display and swipe up to unlock the screen. 

```bash
source launch_lesson5.sh # or launch_lesson7.sh
```

Note: Reboot after every run. 

Note: Do not use a password which will delay the screen unlock.

Note: If `adb` fails due to permission errors, check if you have allowed USB debug access from your machine. You might also need to enable `Use USB to Transfer files` from the swipe-down menu every time the phone reboots. 

## A Note on checkGen

If you have followed all the above instructions, congratulations, you already have a Chromium and Android source tree with all security checks in place. If you wish to port WebGL security checks from Chromium yourself, you can try our open-sourced checkGen tools:

```bash
git clone https://github.com/trusslab/milkomeda_checkgen.git
cd milkomeda_checkgen
# Configurate your Chromium and Android path in gles_patcher.py
# Make sure your Chromium and framework/native does not have any Milkomeda commits
```

Create a copy of GLES2 libs by typing:

```bash
cp -r <Milkomeda home>/framework/native/opengl/libs <Milkomeda home>/framework/native/opengl/medalibs 
```

Now, you are ready to run checkGen:

```bash
python gles_patcher.py
```

Note: The word `yaap` in our code stands for Yet Another Automatic Patcher.

Note: checkGen is not production-ready. The comments in `gles_patcher.py` have more details on how to use the generated code. 

## Known issues and future work

* We have not ported the checks for extension and platform-specific OpenGLES APIs. They are not used in our test cases, but we plan to support them in the future.
* There is a small chance that Milkomeda fails to run. If you see a blank screen with no apparent error in `adb logcat`, please reboot the phone and retry. 
* Running Milkomeda requires the screen to be closed until the launch process is complete. We plan to resolve this issue.

# Acknowledgments

This work (i.e., design and implementation of Milkomeda) was supported by NSF awards #1617513, #1513783, and
ONR award N00014-17-1-2513.
