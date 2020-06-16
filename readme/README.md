# OpenWRT SDK for GL.iNet devices

OpenWRT SDK for GL.iNet devices. The OpenWRT SDK is a pre-compiled environment suitable for creating custom packages without having to compile the entire OpenWRT build environment. 

  ![:!:](https://openwrt.org/lib/images/smileys/icon_exclaim.gif) Do everything as normal user, don't use root user or sudo!

  ![:!:](https://openwrt.org/lib/images/smileys/icon_exclaim.gif) Do not build in a directory that has spaces in its full path 

## System requirements

- x86_64 platform
- Ubuntu or another linux distro

Compiling under Windows can be done using the Windows Subsystem For Linux (WSL) with Ubuntu installed to it. Follow the guide bellow, installing Ubuntu 18.04 LTS from the Microsoft Store: 

 https://docs.microsoft.com/en-us/windows/wsl/install-win10

## Preparing your build environment

To use the SDK on your system will usually require you to install some extra packages.

For **Ubuntu 18.04 LTS**, run the following commands to install the required packages:

```
$ sudo apt update && sudo apt upgrade -y
$ sudo apt install asciidoc bash bc binutils bzip2 fastjar flex gawk gcc genisoimage \ 
gettext git intltool jikespg libgtk2.0-dev libncurses5-dev libssl1.0-dev make \
mercurial patch perl-modules python2.7-dev rsync ruby sdcc subversion unzip util-linux \ 
wget xsltproc zlib1g-dev zlib1g-dev -y
```

# Downloads

```
$ git clone https://github.com/gl-inet/sdk.git
```

The SDK requires a "case sensitive" system, Windows is unfortunately not. To run the SDK in WSL you **MUST** clone the repo to the linux folder tree, ie: `/home/<username>/` or any other folder you choose. This is required, you **CAN NOT** run it from `/mnt/c/` or any other windows native drive mounted in WSL. Running the SDK from a Windows mounted disk will result in a failed build with cryptic messages. 

For ease of use, We store the SDK separately. You can download the specified SDK by the following command.

```
$ ./download.sh 
Usage: 
./download.sh [target]   # Download the appropriate SDK

All available target list:
    ar71xx-1806     # usb150/ar150/ar300m16/mifi/ar750/ar750s/x1200
    ramips-1806     # mt300n-v2/mt300a/mt300n/n300/vixmini
    ipq806x-qsdk53  # b1300/s1300
    mvebu-1907      # mv1000
```

# Compiling
## (1) GL.iNet Util compilation method

We provide a script to compile all software packages with all targets SDK or compile all software packages with a single target SDK. You are freely and quickly compile packages for each platform.

```
$ ./builder.sh 
Usage: 
./builder.sh [option]
command:
    [-a]                # Compile all software packages with all targets.
    [-t] [target]       # Compile packages with single targets.
    [-d] [package_path] # Package path.
    [-v]                # Enable compile log.

All available target list:
    ar71xx-1806     # usb150/ar150/ar300m16/mifi/ar750/ar750s/x1200
    ramips-1806     # mt300n-v2/mt300a/mt300n/n300/vixmini
    ipq806x-qsdk53  # b1300/s1300
    mvebu-1907      # mv1000
```

You can put all your packages to a folder, then run the following command to compile packages for the specified platform,

```
$ ./builder.sh -d [packages_path] -t [target]
```

Or run the following command to compile packages for all platform,

```
$ ./builder.sh -d [packages_path] -a
```

## Demo: Compile the hello_world package

1.Enter the SDK directory of clone GL.iNet and download the SDK of corresponding platform of AR750
```
$ cd sdk
$ ./download.sh ar71xx-1806
```

2.Compile the `example/hello_world` package for the ar71xx-1806 package.The compiled packages will be located in the `sdk/1806/ar71xx/bin/packages/mips_24kc/base` folder
```
$ ./builder.sh -d example/hello_world -t ar71xx-1806
```

1.If you want to compile the hello_world package for all platforms, enter the following command
```
$ ./builder.sh -d example/hello_world -a
```

2.Compiling packages for all platforms for the first time is slow,so it can be done in steps.  

Use `./builder.sh [option]` to download the SDKS for all platforms. Execute again `./builder.sh -d [packages_path] -a`

## (2) Official compilation method

1.Place your package in `sdk/<version>/<target>/package/`, then run:

```
$ cd sdk/<version>/<target>/
$ make package/<package_name>/compile V=s
```

2.For example, to compile the hello_world package for the `ar71xx` target, it will look like this:
The compiled package will be in the `sdk/<version>/<target>/bin/` folder
```
$ cd sdk/1806/ar71xx
$ make package/hello_world/{clean,compile} V=s
```

# Installing

Once the package has been compiled, you can transfer the package via SSH to the GL-iNet device following the guides here:

https://docs.gl-inet.com/en/3/app/ssh/

Then running:
```
$ opkg install <package_name>.ipk
```
Will install the package on the device without internet. 

After the hello_world package above is installed successfully, run the following command
```
$ /mnt/hello_world
```
This will appear on the screen as follows.
```
****************
**Hello world!**
****************
```

# Common mistakes

It is recommended that you use the `V=s` flag at the end of the `make` command, but it is not required. The compiler will print all messages and show if there are any errrors. If you know the packages are all correct, you can omit it for a more clean output.

Some packages require other packages to compile. If your package gives warnings like this:

```
WARNING: Makefile 'package/ddserver/Makefile' has a dependency on 'uclibcxx', which does not exist
```

Before the compilation begins, or when compiling with the `V=s` flag you get an error towards the end like this:

```
Package ddserver is missing dependencies for the following libraries:
libuClibc++.so.0
```

You will need to install the dependency before compiling your package. To do that run:

```
$ ./scripts/feeds update -f
```

This will update the package lists, then run:

```
$ ./scripts/feeds install <dependency_name>
```

For the example above, it required the uClibc++ library, so we install it:

```
$ ./scripts/feeds install uclibcxx
```

Re-run the `make package/<package_name>/compile V=s` command and your package should compile without issues.
 
------

```
To make a custom package from any program you can follow the guides here:
```

https://github.com/mwarning/openwrt-examples

```
And read about the OpenWRT package structure here:
```

https://openwrt.org/docs/guide-developer/packages
