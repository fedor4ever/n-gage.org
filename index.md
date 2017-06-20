# N-Gage Emulation Docs
_Work in Progress by [@loociano](https://github.com/loociano), last update on June 2017_

![ ](https://github.com/loociano/N-Gage-emu-docs/blob/master/ngage-motherboard.png?raw=true)

## Emulators and Tools

* [NGEmu](https://github.com/NGEmu/NGEmu) by [@tambry](https://github.com/tambry): HLE N-Gage emulator written in C++ for Windows
* [Rusty Starship](https://gitlab.com/tambre/rusty-starship) by [@tambre](https://gitlab.com/tambre): Collection of tools

## System Specs

* CPU: [ARM920T](https://en.wikipedia.org/wiki/ARM9#ARM920T) @ 104 MHz
  + ARM9T Technical Reference Manual [PDF](http://www.atmel.com/Images/ARM_920T_TRM.pdf) · [HTML](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.ddi0151c/I1004722.html)
* Memory: 3.4 MB internal memory, MultiMediaCard (MMC) external memory
* OS: [Series 60](https://en.wikipedia.org/wiki/S60_(software_platform)) 1st Edition, Feature Pack 1 (S60 1.2, or Symbian OS 6.1)
* Display: TFT 176x208 pixels, 4096 colors

## S60

* [S60 version comparison (Wikipedia)](https://en.wikipedia.org/wiki/Symbian#Symbian_.28S60.29_version_comparison)
* [S60 versions and supported devices (Wikipedia)](https://en.wikipedia.org/wiki/S60_(software_platform)#Versions_and_supported_devices)
* [Getting Started with C++ Development on the Series 60 SDK (Web Archive, 2002)](https://web.archive.org/web/20050228053950/http://www.symbian.com/developer/techlib/papers/series60/series60.html)

### Downloads 

* [S60 SDK 1.2 for Windows](http://urjaman.ddns.net/sissshare/s60-2ndEd-dev/1stEd/nS60_sdk_v1_2.zip)
  + Minimum requirements: [Java 2 Runtime Environment 1.3.1](http://www.oracle.com/technetwork/java/javasebusiness/downloads/java-archive-downloads-javase13-419413.html), [Microsoft Visual C++ 6.0](https://www.microsoft.com/en-ie/download/details.aspx?id=9183), [Active Perl 5.1.8](https://sourceforge.net/p/wpbdc/website/ci/2ee71367b1932176847e8f969af85168d94c89f4/tree/Download/ActivePerl-5.6.1.635-MSWin32-x86.msi?format=raw)

### Building a S60 Application on Windows

Check that `perl`, `java`, `nmake` and `epoc` commands are working.
```
$ perl -version
$ java -version
$ nmake /HELP
$ epoc
```
To build for the emulator
```
$ cd C:\Symbian\6.1\Series60\Series60Ex\helloworld\group
$ bldmake bldfiles
$ abld build
```
Run the emulator:
```
$ epoc
```
To build for the actual device
```
$ cd C:\Symbian\6.1\Series60\Series60Ex\helloworld\group
$ bldmake bldfiles
$ abld build armi urel
$ cd ..\
$ makesis helloworld.pkg
```

## File Formats

```
.sis = Application Installation file
  .app = Application (E32Image)
  .rsc = Resource binary
  .aif = Icon bitmaps
```

* [E32Image (.app)](http://web.archive.org/web/20070616175615/http://www.antonypranata.com/articles/e32fileformat.html)
* [Multibitmap (.mbm)](http://fileformats.archiveteam.org/wiki/EPOC_MBM)
* [Application Information File (.aif)](http://fileformats.archiveteam.org/wiki/EPOC_AIF)
* [EPOC32 File formats](http://www.koeniglich.de/epoc32_fileformats.txt)
* [Deark](http://entropymine.com/deark/): .mbm to .png converter

Use `deark` to extract images from .aif:
```
$ deark file.aif

Module: epocimage
Format: EPOC AIF
Error: Unsupported bits/pixel (1) for grayscale image
Writing output.000.png
Error: Unsupported bits/pixel (1) for grayscale image
Writing output.001.png
```

Use utility `bmconv` from SDK to extract images from .mbm:
```
$ bmconv /v images.mbm

BMCONV version 103.
images.mbm is a File store containing 3 bitmaps

Bitmap 1 information:
Pixel size 176 x 44
Twips size 352 x 88
24 Bpp Colour
24 bit RLE compression 72%

Bitmap 2 information:
Pixel size 176 x 44
Twips size 352 x 88
24 Bpp Colour
24 bit RLE compression 72%

Bitmap 3 information:
Pixel size 176 x 208
Twips size 263 x 311
24 Bpp Colour
24 bit RLE compression 15%

$ bmconv /u images.mbm 1.bmp 2.bmp 3.bmp

BMCONV version 103.
Decompiling...
Epoc file: images.mbm

Bitmap file 1   : 1.bmp
Bitmap file 2   : 2.bmp
Bitmap file 3   : 3.bmp
Success.
```

### ROM Header

_Extracted from NGEmu_

```
0x0000 UID1
0x0004 UID2
0x0008 UID3
0x000c UID checksum
0x0010 Signature
0x0014 CPU
0x0018 Code checksum
0x001c Data checksum
0x0020 Build (minor)
0x0022 Build (major)
0x0024 Timestamp (msb)
0x0028 Timestamp (lsb)
0x002c Flags
0x0030 Code size
0x0034 Data size
0x0038 Heap minimum size
0x003c Heap maximum size
0x0040 Stack size
0x0044 BSS size
0x0048 Entry point offset
0x004c Code base address
0x0050 Data base address
0x0054 DLL count
0x0058 Export offset
0x005c Export count
0x0060 Text size
0x0064 Code offset
0x0068 Data offset
0x006c Import offset
0x0070 Code relocation offset
0x0074 Data relocation offset
0x007c Priority 
```
Flags:
```
Bit 28-32: Import format
Bit 24-27: Header format
Bit 8-23: ???
Bit 5-7: Entry point type
Bit 3-4: ABI
Bit 2: Fixed address
Bit 1: Call entry point
Bit 0: Executable type
```

#### UIDs

Symbian OS uses Unique IDentifiers (UID) for identifying components. Each component is identified by three 32-bit UID inte-
gers: UID1, UID2 and UID3.

* UID1: specifies the type of component. 
```
0x1000007a = Executable (EXE)
0x10000079 = Library (DLL)
```
* UID2: more specific type of component. It can be zero if UID1 is `EXE`.
* UID3: it is the most specific identifier, must be unique. It can be zero if UID1 is `EXE`.

## References

* [N-Gage (device) - Wikipedia](https://en.wikipedia.org/wiki/N-Gage_(device))
* [N-Gage QD - Wikipedia](https://en.wikipedia.org/wiki/N-Gage_QD)
