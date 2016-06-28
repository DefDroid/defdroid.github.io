---
layout: default
---
**Page still under construction, stay tuned**{: style="color: red"}

### **Warning**{: .label .label-warning}

**DefDroid is a research prototype and is provided on an "AS IS" BASIS, WITHOUT 
WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. In no event shall 
the creators of DefDroid or any of its contributors be liable for any direct, indirect, 
incidental, special, exemplary, or consequential damages (including, but not limited to, 
loss of use, data, or profits; or business interruption) however caused and on any 
theory of liability, whether in contract, strict liability, or tort (including 
negligence or otherwise) arising in any way out of the use of this software, even 
if advised of the possibility of such damage.**
{: .text-danger}

# Guide for building DefDroid

## Generic instructions
The hardware requirements to build DefDroid are roughly the same as building the original
Android release: you need to have a decent machine running 64-bit Linux or Mac OS: 8+ 
GB RAM, 100+ GB storage (preferablly SSD), and fast Internet connection.

### Prepare build environment
Before the actual building, make sure you have JDK 7, and prepare the build environment 
by following the Android [build guide](https://source.android.com/source/initializing.html) 
to install necessary libraries and packages. For example, if your machine is running 
Ubuntu 14.04, run the following commands:

{% highlight bash %}
$ sudo apt-get install git-core gnupg flex bison gperf build-essential \
  zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 \
  lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev ccache \
  libgl1-mesa-dev libxml2-utils xsltproc unzip
{% endhighlight %}

### Install `repo` command
Android release uses a Git wrapper called `repo` to manage the source code. Get it
with

{% highlight bash %}
$ mkdir ~/bin
$ PATH=~/bin:$PATH
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
{% endhighlight %}

### <a name="source_initialize"></a>Initialize source code repository
First, create the workspace to store the source code and build result: 
`mkdir -p ~/android/defdroid`. In the following, we assume the workspace is in 
`~/android/defdroid`. If you chose a different path, make sure to substitute it
accordingly.

{% highlight bash %}
$ cd ~/android/defdroid
$ repo init -u <repo_url> -b <branch> 
$ repo sync
{% endhighlight %}

Replace `<repo_url>` and `<branch>` with the desired Android base repo and branch
that DefDroid has been ported to:

{:.table .table-bordered .table-hover}
| Release | Version | `<repo_url>` | `<branch>` |
| ------- | ------- | ------------ | --------- |
| [CM](http://www.cyanogenmod.org) | 11.0 | `https://github.com/CyanogenMod/android.git` | `cm-11.0` |
| [AOSP](https://source.android.com/index.html) | 4.4 | `https://android.googlesource.com/platform/manifest` | `android-4.4.4_r1` |
| [AOSP](https://source.android.com/index.html) | 5.1 | `https://android.googlesource.com/platform/manifest` | `android-5.1.1_r3` |

The last step (`repo sync`) would take a while to download the repository.

### Enable `ccache`
To speed up the building process on re-compilation, it is recommended to enable
`ccache`:

{% highlight bash %}
$ export USE_CCACHE=1
$ export CCACHE_DIR=<path-to-your-cache-directory>
$ prebuilts/misc/linux-x86/ccache/ccache -M 50G
{% endhighlight %}

The `CCACHE_DIR` setting by default is in `~/.ccache`.


## Build for [CM](http://www.cyanogenmod.org) 11.0

### Source code repository customization
The repo command to be used in the [initialization step](#source_initialize) for CM 
11.0 build is `repo init -u https://github.com/CyanogenMod/android.git -b cm-11.0`.
Then get the DefDroid modifications by using the local manifests approach:

{% highlight bash %}
$ mkdir .repo/local_manifests
$ git clone git@github.com:DefDroid/defdroid_local_cm_falcon.git .repo/local_manifests
$ repo sync
{% endhighlight %}

### Get prebuilt apps in CM
This step only needs to be done once to populate the built-in apps from CM into the
repository:

{% highlight bash %}
$ cd ~/android/defdroid/vendor/cm
$ ./get-prebuilts
{% endhighlight %}

### Prepare device-dependent code and blobs
To build a working image for a particular device usually requires device-dependent code
and proprietary blobs to be pulled in the repo. In CM release, this is mainly handled 
with `breakfast` command and some scripts. For example, to build for Motorola G device, 
you need to run:

{% highlight bash %}
$ cd ~/android/defdroid
$ source build/envsetup.sh
$ breakfast falcon
{% endhighlight %}

Then you need to pull the proprietary blobs from the device: connect your Motorola G 
device to your computer via USB; make sure the `adb` command from Android SDK is in 
your PATH; and ironically, your Motorola G should already be running a CM release
for the following script to work:
{% highlight bash %}
$ cd ~/android/defdroid/device/motorola/falcon
$ ./extract-files.sh
{% endhighlight %}

### Finally, build!
Using Motorola G target as an example:

{% highlight bash %}
$ croot
$ brunch falcon
{% endhighlight %}

# Install DefDroid
Once the system finishes building, you should see the output image in 
`out/target/product/<device>/cm-11-<data>-UNOFFICIAL-<device>.zip` where 
`<device>` is the code name for your Android devices. For Motorola G, it's 
`falcon`.

With the image zip file, follow the standard way to flash it to the phone.
If you are not familiar with how to flash a custom ROM, there are plenty
of guides online. For example, for the Motorola G device, you can follow
steps in [this guide](http://wiki.cyanogenmod.org/w/Install_CM_for_falcon)


