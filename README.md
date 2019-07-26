AptioFix
========

[![Build Status](https://travis-ci.com/acidanthera/AptioFixPkg.svg?branch=master)](https://travis-ci.com/acidanthera/AptioFixPkg) [![Scan Status](https://scan.coverity.com/projects/16009/badge.svg?flat=1)](https://scan.coverity.com/projects/16009)

AptioFixPkg drivers fixing certain UEFI APTIO Firmware issues relevant to booting macOS.

**WARNING**: The code in this repository should be considered to be a proof of concept draft quality, and is only intended to be used as a software implementation guide. Due to the lack of time, this codebase may contain partially understood reverse-engineering samples, almost no documentation, hacks, and absolute ignorance of EDK2 codestyle.

## AptioMemoryFix

Fork of the original [OsxAptioFix2](https://sourceforge.net/p/cloverefiboot/code/HEAD/tree/OsxAptioFixDrv/) driver with a cleaner (yet still terrible) codebase and improved stability and functionality.

**Important Notes**:
To debug boot.efi errors on 10.13, aside the usual verbose (-v) boot-arg, you may need a boot.efi [patch](http://www.insanelymac.com/forum/topic/331381-aptiomemoryfix/page-7#entry2572595) or an nvram preference. Either is necessary to get sensible error messages instead of 'does printf work??', but the patch may help in case you have issues when reading nvram.

Before using AptioMemoryFix please ensure that you have:
- Most up-to-date UEFI BIOS firmware (check your motherboard vendor website).
- Fast Boot and Hardware Fast Boot disabled in BIOS if present.
- Above 4G Decoding or similar enabled in BIOS if present.  
**WARNING**: On some motherboards (notably ASUS WS-X299-PRO) this option causes adverse effects, and must be disabled. While no other motherboards with the same issue are known, consider this option to be first to check if you have erratic boot failures.
- VT-d disabled in BIOS if present (you could also drop ACPI DMAR table with a bootloader).
- _No_ `slide` boot argument present in NVRAM or anywhere else (it is not necessary unless you cannot boot at all or see `No slide values are usable! Use custom slide!` message from AptioMemoryFix).
- CFG Lock (MSR 0xE2 write protection) disabled in BIOS if present (consider [patching it](https://github.com/LongSoft/UEFITool/blob/master/UEFIPatch/patches.txt) otherwise if you have enough skills). See VerifyMsrE2 section for more details.
- CSM disabled in BIOS if present (you may need to flash GOP ROM on NVIDIA 6xx/AMD 2xx or older, using [GopUpdate](https://www.win-raid.com/t892f16-AMD-and-Nvidia-GOP-update-No-requests-DIY.html#msg15730) or [AMD UEFI GOP MAKER](http://www.insanelymac.com/forum/topic/299614-asus-eah6450-video-bios-uefi-gop-upgrade-and-gop-uefi-binary-in-efi-for-many-ati-cards/page-1#entry2042163) could simplify things).
- EHCI/XHCI Hand-off enabled in BIOS *only* if boot stalls unless USB devices are disconnected.
- VT-x, Hyper Threading, Execute Disable Bit enabled in BIOS if present.
- While it may not be required, sometimes you have to disable Thunderbolt support, Intel SGX, and Intel Platform Trust in BIOS if present.
When debugging sleep issues you may want to (temporarily) disable Power Nap and automatic power off, which appear to sometimes cause wake to black screen or bootloop issues on older platforms. The particular issues may vary, but in general you should check ACPI tables first. Here is an example of a bug found in some [Z68 motherboards](http://www.insanelymac.com/forum/topic/329624-need-cmos-reset-after-sleep-only-after-login/#entry2534645). To turn Power Nap and the others off run the following commands in Terminal:
    - `sudo pmset autopoweroff 0`
    - `sudo pmset powernap 0`
    - `sudo pmset standby 0`

    Note, that these settings may reset at hardware change and in certain other circumstances. To view their current values run `pmset -g`.

Note, if you see a lot of Start/End lines in verbose mode you must ensure you do not have `bootercfg` variable stored in NVRAM. See [this post](https://www.insanelymac.com/forum/topic/331381-aptiomemoryfix/?page=6&tab=comments#comment-2572819) for details. If it still does not help you could use the following command in Terminal (either with System Integrity Protection disabled or from Recovery HD):
```
sudo nvram bootercfg="log=0 debug=0"
```

#### Features (compared to the original)
- Automatically finds the available memory region for boot.efi to use
- Implements KASLR support for systems where certain slides cannot be used
- Adds Safe Mode support on systems with used lower addresses
- Ensures no slide argument leak to the operating system
- Attempts to more properly handle the memory mappings
- Implements read-only and write-only RT variables beneficial to Lilu plugins
- Works around missing gUnicodeCollation and `PlatformLang` on several firmwares
- Fixes broken legacy hash services protocol (only MD5, SHA-1, SHA-256 are supported)
- Ensures that GetTime does not report unspecified timezone
- Supports boot.efi chainloading (e.g. via `sudo nvram recovery-boot-mode=open`)
- Implements new mapping code when waking from hibernation (still not stable enough)

#### Credits
- [Apple](https://www.apple.com) for macOS
- [AMI](https://ami.com) for APTIO firmware
- [dmazar](https://sourceforge.net/u/dmazar/profile/), [apianti](https://sourceforge.net/u/apianti/), [CodeRush](https://github.com/NikolajSchlej), [night199uk](https://github.com/night199uk), [Slice](https://sourceforge.net/u/slice2009/) for developing the original OsxAptioFix driver
- [Download-Fritz](https://github.com/Download-Fritz) for implimenting some of this code and invaluable suggestions
- [vit9696](https://github.com/vit9696) for all the mess
- Big thanks go to all the contributors and researchers involved in AMI APTIO exploration!
- Additional thanks go to people on [AppleLife](http://applelife.ru) and [InsanelyMac](http://insanelymac.com) who helped to test things!
