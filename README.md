**This is [toaonly's fork](https://github.com/toaonly/AltServer-Linux) (branch `new`), based on jaakkopalvaila's `ng` fixes for iOS 26.4+/27 signing, the 2026 Apple ID sign-in block, anisette v3 and netmuxd >= 0.3. It also documents a workaround for the AltStore in-app sign-in HTTP 503. See [README-NG.md](README-NG.md).**

# AltServer-Linux
AltServer for AltStore, but on-device

## Usage

- Install IPA: `./AltServer -u [UDID] -a [AppleID account] -p [AppleID password] [ipaPath.ipa]`
- Running as AltServer Daemon: `./AltServer`
- Full usage (maybe outdated, refer to `./AltServer -h` for the newest):
```
Usage:  AltServer-Linux options [ ipa-file ]
  -h  --help             Display this usage information.
  -u  --udid UDID        Device's UDID, only needed when installing IPA.
  -a  --appleID AppleID  Apple ID to sign the ipa, only needed when installing IPA.
  -p  --password passwd  Password of Apple ID, only needed when installing IPA.
  -d  --debug            Print debug output, can be used several times to increase debug level.

The following environment var can be set for some special situation:
  - ALTSERVER_ANISETTE_SERVER: Set to custom anisette server URL
          if not set, the default one: https://new.ani.sidestore.io, is used
  - ALTSERVER_NO_SUBSCRIBE: (*unused*) Please enable this for usbmuxd server that do not correctly usbmuxd_listen interfaces
```

## Download

- This fork has no Releases yet. The "Build AltServer" GitHub Actions workflow runs on every push and uploads AltServer-{x86_64,aarch64,armv7,i586} artifacts (they expire after 90 days): [Actions](https://github.com/toaonly/AltServer-Linux/actions)
- Prebuilt binaries of the same code: [jaakkopalvaila/AltServer-Linux Releases](https://github.com/jaakkopalvaila/AltServer-Linux/releases) (ng-2026-11-05)

## TODO / Special Features
- [x] Track upstream (AltServer-Windows) develop branch (i.e. Beta version)
- [x] Support Offline Anisette Data Generation (i.e. without Sideloadly)
  - The default is now the anisette v3 server https://new.ani.sidestore.io. A custom `ALTSERVER_ANISETTE_SERVER` must provide Mac anisette (see [README-NG.md](README-NG.md), Fix 5)
  - Old approach: run [alt_anisette_server](https://hub.docker.com/r/nyamisty/alt_anisette_server) & use `ALTSERVER_ANISETTE_SERVER` to specify custom server URL
- [x] Support Wi-Fi Refresh
  - [netmuxd](https://github.com/jkcoxson/netmuxd) now supports network devices (needs netmuxd >= 0.3, see [README-NG.md](README-NG.md), Fix 3)
    - Download `netmuxd`, stop the original `usbmuxd`, and run `netmuxd` before running `AltServer-Linux`
    - ~If netmuxd does not work, please try using special env var `ALTSERVER_NO_SUBSCRIBE`. Enabling this would disable **auto-refresh when plugged-in** of USB devices~

----

## Advanced: Build Instruction (check Github Actions if you cannot build)

- Preparation: `git clone --recursive https://github.com/toaonly/AltServer-Linux`

- Install dependencies (see notes below): corecrypto_static, cpprestsdk static lib, boost static lib

- Build:
  ```
  cd AltServer-Linux
  mkdir build
  make -f ../Makefile -j3
  ls AltServer-*
  ```

- My own build note for you 
  ```
    1. Run alpine docker (change --platform to corresponding architecture you want): 
        docker run --platform=linux/arm/v7 --name altserver-builder-alpine-armv7 -it alpine:3.15 
    2. Install dependencies:
        apk add zsh git curl wget g++ clang boost-static ninja boost-dev cmake make sudo bash vim libressl-dev util-linux-dev zlib-dev zlib-static
    3. Install corecrypto
        download corecrypto from apple website, unzip corecrypto.zip; cd corecrypto; mkdir build; cd build; CC=clang CXX=clang++ cmake ..;
        vim CMakeFiles/Makefile2, delete line starts with "all: corecrypto_perf/....." and "all: corecrypto_test/.....", then make; make install
    4. Install cpprestsdk
        git clone --recursive https://github.com/microsoft/cpprestsdk; cd cpprestsdk; mkdir build; cmake -DBUILD_SHARED_LIBS=0 ..; make; make install
	    (if you're compiling for armv7, you have to grep -Wcast-align, and remove it, or the compiling would fail)
    5. Install libzip
        git clone https://github.com/nih-at/libzip; cd libzip; mkdir build; cd build; cmake -DBUILD_SHARED_LIBS=0 ..; make; make install
    6. Compile AltServer-Linux
        git clone --recursive https://github.com/toaonly/AltServer-Linux
        cd AltServer-Linux
        make -f ../Makefile -j3
	    (if you're compiling for ARM, i.e. armv7 or aarch64, you'll have to remove the -mno-default flag in Makefile)

  ```
