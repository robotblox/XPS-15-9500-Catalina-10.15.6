# XPS-15-9500-Catalina-10.15.6
A work in progress of getting OS X Catalina (10.15.6) to run on a Dell XPS 15 9500 using Opencore.

![Image of About This Mac](https://i.imgur.com/amM5AHA.jpg)

## Introduction
The EFI included in this repository was able to get Mac OS X Catalina 10.15.6 running on a Dell XPS 15 9500, albeit with many issues that will be reviewed in this guide. The specific laptop that this was tested on was the 3840x2400 touchscreen variant with a 1TB PC611 SK Hyinx NVME SSD. This EFI was mainly used to get into the Mac OS X installer and get it installed onto my hard drive, so I haven't done much post install work.

If you would like to get started with creating a Hackintosh on your XPS 9500 but have no experience, I would highly reccomend following Dortania's fantastic [Opencore Install guide](https://dortania.github.io/OpenCore-Install-Guide/) and then returning here for troubleshooting.


## Specs

| Key                    | Value                                                        |
| ---------------------- | ------------------------------------------------------------ |
| CPU                    | Intel Core i7 10750H                                         |
| GPU                    | Intel UHD Graphics 630                                       |
| Builtin Screen         | 15.6" UHD+ IPS Touch Display                                 |
| RAM                    | 32GB DDR4 2933MHz (2x16GB)                                   |
| Internal SSD - Windows | SK Hyinx PC611 1TB NVME SSD                                  |
| External SSD - macOS   | WD Blue™ 3D NAND SATA M.2 2280 SSD 500GB                     |
| Audio                  | Unknown                                                      |
| Wireless               | N/A (intend to use ASUS AC53 USB when USB ports work)        |


Quick Note: My serial number, MLB, and UUID have been removed from the config.plist. Please use CorpNewt's [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) to create your own


## What Works

iGPU: Works normally

Keyboard: Works normally using VoodooPS2

Trackpad & Touchscreen: Sometimes (read below)


## What Doesn't Work

### Trackpad & Touchscreen
The trackpad & touchscreen operate with SSDT-XOSI and the necessary patches under ACPI + VoodooI2C + VoodooI2CHID.

It works nominally after booting into Mac OS X. Anywhere between 30 seconds - 5 minutes after booting into Mac OS however, the trackpad and touchscreen both become completely non-fuctional. If I move my fingers around on the trackpad vigorously, the cursor might move a little bit without animating it's movement, but overall it's unusable and requires a restart of the OS. The keyboard works fine during this time.

I attempted to manually create an SSDT-GPIO as per the Opencore Dortania guide, but with that method the trackpad and touchscreen were never detected.

See also: as recommended by u/pwndupure on Reddit, [this edit](https://github.com/VoodooI2C/VoodooI2C/issues/319#issuecomment-646437623) to VoodooI2C & VoodooI2CHID may work to fix this issue.

I have not been able to try this edit as I don't have access to a functional Mac with Xcode, and therefore I can't build the new kexts. If anyone gives this a shot and it works, please let me know.


### USB Ports (Using a USB-C to USB-A adapter)
I have USBInjectAll.kext with this setup, but USB ports still do not work. When booting into Mac OS, the only USB that is detected regardless of how many times it is ejected and reinserted is the BOOT USB that I have Opencore installed on. 

Hackintool does not detect USBs. CorpNewt's USB tool gives the error: **Was unable to locate any valid ports. Please ensure that you have XHC/EH01/EH02 in your IOReg** (Perhaps we need an ACPI Rename? According to the Dortania Opencore guide, Comet Lake does not require ACPI Renames when USB Mapping, so I'm not sure about this one)

### Audio & Brightness Control

Haven't looked into these two very much yet. I am using the boot-arg aclid=30 with AppleALC, but audio does not work.


## What you need to know that isn't mentioned in Dortania's Opencore guide:

1. SATA Mode in BIOS must be set to `AHCI`. This **will cause a kernel panic** when booting into the Mac OS X USB installer.

- To fix this, you must use the `SSDT-DNVME.aml` file provided in this EFI and put it in your ACPI folder. This was sourced from geek5nan's XPS 7590 Hackintosh EFI, and disables any NVME SSD installed in your system when booting into Mac OS X.

2. `SetupVirtualMap` under Booter > Quirks must be set to **false**. If it is not, there **will be a kernel panic.**

3. iGPU acceleration does not work using the base instructions provided in Dortania's Opencore guide. You will get stuck at **IOConsoleUsers: gIOScreenLockState 3** indefinitely unless you boot with `-igfxvesa` (however, this disables iGPU Acceleration). 

- I used the framebuffer patches found under `PciRoot(0x0)/Pci(0x2,0x0)`, and boot-args found under `NVRAM ` from geek5nan's XPS 7590 config.plist to enable iGPU Acceleration and it worked.


## Conclusion

I intend for this EFI to be a starting point for others to experiment getting Mac OS X running on their Dell XPS 15 9500s with full functionality. All of the discoveries I made to get to this point were a result of countless hours of reading along with trial and error. I am by no means an expert, and need lots of help to get this project off the ground and functional. Thanks for giving this a read, and good luck!


