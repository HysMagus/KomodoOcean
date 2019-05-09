# Scribbles of madman

This branch is planned to be universal for all 3 OS (Linux, Windows, MacOS).
## todo

- Build subsystem: after build win version, if we want to build linux version for example with `build-linux.sh`, univalue and cryptoconditions dirs needs to be cleaned with `make clean` from previous build. otherwise it will not be linked
- Check Mac build with static libs
- Remove unneeded `build-*.sh` files
- Check *.deb package build

## dev notes

### part 1

This file contains only **dev** notes. It's a some kind of notepad or "scribbles of madman" ...

- https://github.com/bitcoin/bitcoin/pull/3914/files - Build fully static versions of bitcoind and bitcoin-cli for older Linux distros
- ```./configure --enable-glibc-back-compat --prefix=$(pwd)/depends/x86_64-pc-linux-gnu LDFLAGS="-static-libstdc++"```
- https://bitcointalk.org/index.php?topic=1080289.0;all -  [HOWTO] compile altcoin for windows on linux using mxe and mingw


```
objdump -T src/qt/komodo-qt | grep 2.25
0000000000000000      DF *UND*	0000000000000000  GLIBC_2.25  __explicit_bzero_chk
0000000000000000  w   DF *UND*	0000000000000000  GLIBC_2.25  getentropy
```

- https://stackoverflow.com/questions/44488972/static-libstdc-works-on-g-but-not-on-pure-gcc
- https://thecharlatan.github.io/GLIBC-Back-Compat/ - Make binaries portable across linux distros

```
-Wl,--wrap=log2f

AX_CHECK_LINK_FLAG([[-Wl,--wrap=__divmoddi4]], [COMPAT_LDFLAGS="$COMPAT_LDFLAGS -Wl,--wrap=__divmoddi4"])
AX_CHECK_LINK_FLAG([[-Wl,--wrap=log2f]], [COMPAT_LDFLAGS="$COMPAT_LDFLAGS -Wl,--wrap=log2f"])
```

- https://drewdevault.com/2016/07/19/Using-Wl-wrap-for-mocking-in-C.html

### part 2

MacOS static build now produce the following binary:

```
src/qt/komodo-qt:
	/usr/local/opt/gcc@6/lib/gcc/6/libstdc++.6.dylib (compatibility version 7.0.0, current version 7.22.0)
	/System/Library/Frameworks/DiskArbitration.framework/Versions/A/DiskArbitration (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/IOKit.framework/Versions/A/IOKit (compatibility version 1.0.0, current version 275.0.0)
	/System/Library/Frameworks/Foundation.framework/Versions/C/Foundation (compatibility version 300.0.0, current version 1570.15.0)
	/System/Library/Frameworks/CoreServices.framework/Versions/A/CoreServices (compatibility version 1.0.0, current version 944.3.0)
	/System/Library/Frameworks/AppKit.framework/Versions/C/AppKit (compatibility version 45.0.0, current version 1671.40.118)
	/System/Library/Frameworks/ApplicationServices.framework/Versions/A/ApplicationServices (compatibility version 1.0.0, current version 50.1.0)
	/System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation (compatibility version 150.0.0, current version 1570.15.0)
	/System/Library/Frameworks/Security.framework/Versions/A/Security (compatibility version 1.0.0, current version 58286.251.4)
	/System/Library/Frameworks/SystemConfiguration.framework/Versions/A/SystemConfiguration (compatibility version 1.0.0, current version 963.250.1)
	/System/Library/Frameworks/CoreGraphics.framework/Versions/A/CoreGraphics (compatibility version 64.0.0, current version 1251.12.0)
	/System/Library/Frameworks/OpenGL.framework/Versions/A/OpenGL (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/AGL.framework/Versions/A/AGL (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/Carbon.framework/Versions/A/Carbon (compatibility version 2.0.0, current version 158.0.0)
	/System/Library/Frameworks/CoreText.framework/Versions/A/CoreText (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/LDAP.framework/Versions/A/LDAP (compatibility version 1.0.0, current version 2.4.0)
	/usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 400.9.4)
	/usr/local/opt/gcc@6/lib/gcc/6/libgomp.1.dylib (compatibility version 2.0.0, current version 2.0.0)
	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.250.1)
	/usr/local/lib/gcc/6/libgcc_s.1.dylib (compatibility version 1.0.0, current version 1.0.0)
	/System/Library/Frameworks/CFNetwork.framework/Versions/A/CFNetwork (compatibility version 1.0.0, current version 978.0.7)
	/System/Library/Frameworks/ImageIO.framework/Versions/A/ImageIO (compatibility version 1.0.0, current version 1.0.0)
	/usr/lib/libobjc.A.dylib (compatibility version 1.0.0, current version 228.0.0)
```

To avoid dynamic linking from `/usr/local/lib/gcc/6/` need to add `-static-libgcc` linker flag, but binary compiled and linked this way with gcc6 produced some exceptions right after launch, probably due this:

https://stackoverflow.com/questions/50920999/why-dont-exceptions-work-with-gcc7-and-static-libgcc-on-osx

So, the solution is build with gcc6, but link with gcc8, like `../libtool  --tag CXX  --mode=link g++-8 -g -O2 <...> -static-libgcc` .