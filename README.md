# Unpacking Android Builds
This document serves as a reference on how to unpack Android builds to extract its DEX, APK, and JAR files which contain the source code of Android's application framework and the system apps that would be installed on the target device. It is not meant for explaining the logic underlying the packing/unpacking techniques.

## Sources of Android Builds
I'm aware of the following sources where you can retrieve Android builds. Please, feel free to suggest more.
- Google AOSP (master builds): https://ci.android.com/
- Google Factory Images: https://developers.google.com/android/images
- Samsung: https://www.sammobile.com/firmwares/

## Requirments

Please note that not all requirments are needed. Use them as you go through out the instructions.

 - [**simg2img**](https://github.com/anestisb/android-simg2img) to convert sparse images to raw images that can be mounted
 - [**Vdex Extractor**](https://github.com/anestisb/vdexExtractor) to decompile and extract Android dex bytecode from vdex files
 - [**Compact Dex Converter**](https://github.com/anestisb/vdexExtractor#compact-dex-converter) to convert cdex files to standard dex 
 - [**unlz4**](http://manpages.ubuntu.com/manpages/bionic/man1/lz4.1.html) to decompress lz4 files

## Extracting The System Image

Please note that some of the following steps is optional. Read the details to know when to execute each step.

### 1) Locating system.img/super.img file

First thing to do after downloading the Android build, is to decompress the downloaded file(s) and locate the `system.img` or `super.img` file. For ease of discussion, we will call this file `system.img` from here on. In SAMSUNG builds, this file can be compressed using `lz4`. You can use  `lz4` utility or any other compatable tool to decompress the file.

    unlz4 system.img.lz4

#### 2) Converting sparse image into raw image

Some `system.img` files are sparse and require one extract step. According to [this](https://unix.stackexchange.com/a/612890) post, the image is sparse if the output of the following command is `< 1`. 

    find system.img -type f ! -size 0 -printf '%S\n'

If the image is sparse, use the `simg2img` utility to convert it into a raw image, execute the following command.

    simg2img ./system.img system.img.raw

#### 3) Mounting the system image

Create a directory to mount the the system image and mount it

    mkdir ./system
    mount -o loop -t ext4 ./system.img.raw ./system
 
 If this step fails to due to:

     mount: {...}system: wrong fs type, bad option, bad superblock on /dev/loop, missing codepage or helper program, or other error.

Then head to the following step.

#### 4) Extracting the system image

Change the extension of the system image to .zip and extract it manually.


## Extacting DEX files

Depending on the Android version of the build, the DEX files might be packed inside VDEX files. To extract the DEX file from a file named `services.vdex`, execute the following command and make sure that the corresponding binaries are in the `PATH`.

    vdexExtractor -i services.vdex -o ./output_dir --deps -f

In case the output files are in the CDEX format, execute the following command to convert them to standard DEX format. 

    compact_dex_converter services.cdex

## Final Words

If any of the above didn't work for you, feel free to create a ticket and share the build image you are trying to unpack and let's figure it out together.
