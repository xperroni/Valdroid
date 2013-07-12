Valdroid 3.8.1 / r8e
====================

**Valdroid** is a binary build of [Valgrind](http://valgrind.org/) to Android on ARMv7.

This version was built from Valgrind 3.8.1 using the Android NDK r8e toolchain on Linux 32 bits.

Valdroid is meant as a convenience to Android NDK developers working with plain native executables, who might be looking for a quick setup in order to profile their applications. However I provide **no guarantees** this will work for you, other than that I use it myself and it does work for me. It goes without saying **I am not in any way associated to Valgrind,** I'm just a random user sharing a solution to a problem I had. In case of problems I am *not* the person to ask for help, all I did was follow the steps from Valgrind's [Android README](http://valgrind.org/docs/manual/dist.readme-android.html) with minor variations; if it doesn't work your guesses are as good as mine, and you'd better contact the [Valgrind community](http://valgrind.org/support/summary.html) for support.

That said I have tried to make this manual reasonably sound and document all pitfalls I met along the way; if something else pops up on future releases, I'll make a point to keep it updated. Also if you find a problem not described here and figure a solution to it, I'd appreciate if you could share it. I am happy to pass along solutions to problems I have already faced, just don't expect me to bother with your problem if it isn't my problem too.

Usage
-----

**Note:** *I have only ever used Valgrind on rooted Android devices, and have not bothered looking into whether the steps below (or some variation thereof) would work otherwise. As you're looking into profiling a plain native app on Android I figure you must be savvy enough to find your way around if it doesn't, but be warned.*

Clone the repository, open a command prompt to its base folder and type the commands below to copy Valdroid to your Android device:

    adb shell mkdir -p /data/local/valdroid
    cd valdroid
    adb push . /data/local/valdroid
    adb shell chmod 777 /data/local/valdroid/bin/*
    adb shell chmod 777 /data/local/valdroid/lib/valgrind/*-arm-linux

Now, assuming your application `foo` is in folder `/data/local` and you want to use `callgrind` to profile its runtime performance, the command below will start it in profile mode with initial argument `--bar=fubar`:

    adb shell /data/local/valdroid/bin/valgrind \
        --callgrind-out-file=/data/local/callgrind.out \
        --tool=callgrind /data/local/foo --bar=fubar

During application startup you may see the message below:

    --XXXX-- WARNING: Serious error when reading debug info
    --XXXX-- When reading debug info from /data/local/valdroid/lib/valgrind/vgpreload_core-arm-linux.so:
    --XXXX-- Can't make sense of .data section mapping

I have no idea what it means, but the profiler seems to work nevertheless, so I haven't bothered looking into it. I'd appreciate a clarification if you happen to know what it's about.

Profile data will be recorded to file `/data/local/callgrind.out` and can be recovered with the command below:

    adb pull /data/local/callgrind.out .

Build from Source
-----------------

As I mentioned before Valgrind's [Android README](http://valgrind.org/docs/manual/dist.readme-android.html) is the authoritative reference on how to build Valgrind for Android. I have however introduced my own variations to the process, which I document below.

*As already noted I have used* [version r8e](http://dl.google.com/android/ndk/android-ndk-r8e-linux-x86.tar.bz2) *of the Android NDK for this build. I have no idea whether the steps below will work for other versions.*

Download the [Android NDK](http://developer.android.com/tools/sdk/ndk/index.html) toolchain and extract it to `$HOME/bin`, then run the commands below to create a new stand-alone toolchain:

    mkdir -p $HOME/bin/android-14-ndk-4.7
    $HOME/bin/android-ndk-r8e/build/tools/make-standalone-toolchain.sh \
        --toolchain=arm-linux-androideabi-4.7 --platform=android-14 \
        --install-dir=$HOME/bin/android-14-ndk-4.7

Download Valgrind's [source archive](http://valgrind.org/downloads/) and extract it. Open a prompt to the extracted folder and type the commands below:

    NDK=$HOME/bin/android-14-ndk-4.7
    BIN=$NDK/bin
    SYS=$NDK/sysroot
    AR=$BIN/arm-linux-androideabi-ar
    CC="$BIN/arm-linux-androideabi-gcc --sysroot=$SYS"
    CXX="$BIN/arm-linux-androideabi-g++ --sysroot=$SYS"
    LD=$BIN/arm-linux-androideabi-ld

    ./autogen.sh

    ./configure CC="$CC" CXX="$CXX" CPPFLAGS="-DANDROID_HARDWARE_generic" \
        --prefix=/data/local/valdroid --host=armv7-unknown-linux \
        --target=armv7-unknown-linux --with-tmpdir=/sdcard

    make -j2
    make -j2 install DESTDIR=`pwd`/android

If everything works, compiled files will be written to `android/data/local/valdroid` under the base source folder.

If you get errors about undefined number types (`uint32_t` etc), open header file `$HOME/bin/android-14-ndk-4.7/sysroot/usr/include/elf.h`, add a line `#include <sys/types.h>` before `#include <sys/exec_elf.h>` (somewhere around line 55), clean the project with `make clean` and try again. If the thought of fiddling with toolchain-wide headers bothers you, know that I completely understand how you feel, and I don't care.
