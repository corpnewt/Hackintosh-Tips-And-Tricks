# Hackintosh-Tips-And-Tricks

I'm going to try and make this readable - and hopefully add/update/change info as I get it, but no promises :)

Click the "\[Return\]" links to return to the Index.

##Index

* [OSX Boot Issues](#osx-boot-issues)
 * [Missing Bluetooth Controller Transport](#missing-bluetooth-controller-transport)
 * [Enable Legacy Matching](#enable-legacy-matching)
 * [Prohibited Sign](#prohibited-sign)
 * [OsxAptioFix Errors](#osxaptiofix-errors)
 * [Row Of Plus Signs](#row-of-plus-signs)
* [Kernel Panics](#kernel-panics)
 * [AppleIntelCPUPowerManagement](#appleintelcpupowermanagement)
 * [BluetoothHostControllerUARTTransport](#bluetoothhostcontrolleruarttransport)
* [Audio](#audio)
 * [Skylake](#skylake)
 * [AppleALC](#applealc)
 * [HDMI Audio](#hdmi-audio)
* [Hardware Exceptions](#hardware-specific-tips)
 * [Asus Z170-A](#asus-z170-a)
* [Tools](#tools)
 * [Mounting the EFI Partition](#mounting-the-efi-partition)
 * [Repairing Permissions and Rebuilding Kext Cache](#repairing-permissions-and-rebuilding-kext-cache)

-

##OSX Boot Issues

###Missing Bluetooth Controller Transport

This seems like a Bluetooth error - but it's actually just the last line before the GPU initialization starts.  It usually means the GPU isn't initializing.

####NVIDIA

* **Maxwell - 900 Series (also includes 745, 750, 750ti)**
 * Ensure you have NVIDIA Injection **disabled** in Clover
 * If you have already installed the NVIDIA Web Drivers - boot with `nvda_drv=1`
 * If not - boot with `nv_disable=1`
* **Kepler - GTX 600 & 700 Series (excludes 745, 750, 750ti)**
 * Ensure you have NVIDIA Injection **disabled** in Clover
 * Remove `nv_disable=1` and `nvda_drv=1` from boot arguments if present
 * If you still can't get in - try using `nv_disable=1`, installing Web Drivers, then using `nvda_drv=1`
 * Some of the Kepler cards work better with the Web Drivers
* **Fermi**
 * Ensure you have NVIDIA Injection **disabled** in Clover
 * I believe most of these work OOB - but may suffer from stuttering
 * If your stutters, try using `nv_disable=1`, installing Web Drivers, then using `nvda_drv=1`
 
####Intel
 
* **HD 4600 - Haswell**
 * Ensure Intel Injection is **enabled** in Clover
 * Remove any *ig-platform-id*
 * Set primary display output to IGPU in BIOS
* **HD 530 - Skylake**
 * Ensure Intel Injection is **enabled** in Clover
 * Set *ig-platform-id* to *0x19120000*
 * Set primary display output to IGPU in BIOS

[\[Return\]](#index)

###Enable Legacy Matching

This is something I've seen on *some* Skylake builds.  It's related to USB ownership.

To fix it - enable *FixOwnership* in config.plist (you may also need to enable *Inject*):

        <key>USB</key>
        <dict>
            <key>FixOwnership</key>
            <true/>
            <key>Inject</key>
            <false/>
        </dict>
        
[\[Return\]](#index)

###Prohibited Sign

Shows up usually when booting the USB drive - the text gets garbled, a cirlce with a slash through it shows up, and the line *Still waiting for root device...* can be seen at the end.

This usually means that the USB drive got "lost" as the OS is booting.  The most commonly accepted solution is to try *all* USB ports on your board.

USB 2.0 typically work better (and even better still when combined with a USB 2.0 flash drive).

You may need to inject all your USB ports via RehabMan's [USBInjectAll.kext](https://bitbucket.org/RehabMan/os-x-usb-inject-all/downloads), and add the following to your config.plist -> KextsToPatch:

            <dict>
                <key>Comment</key>
                <string>change 15 port limit to 30 in AppleUSBXHCIPCI</string>
                <key>Disabled</key>
                <false/>
                <key>Find</key>
                <data>
                g72M/v//EA==
                </data>
                <key>Name</key>
                <string>AppleUSBXHCIPCI</string>
                <key>Replace</key>
                <data>
                g72M/v//Hw==
                </data>
            </dict>

[\[Return\]](#index)

###OsxAptioFix Errors

Memory mapping errors are tricky - if you're on an X99 system, you may be out of luck, although I *do* believe there are some custom *OsxAptioFix* drivers out there - reports on their success are hit and miss.

For everyone else - this usually means that you need to swap *OsxAptioFixDrv-64.efi* for *OsxAptioFix****2****Drv-64.efi* in your EFI partition at */EFI/CLOVER/drivers64UEFI/* (**do not** have both fixes in that folder - that's like Russian Roulette with memory mapping).

[\[Return\]](#index)

###Row Of Plus Signs

Usually attributed to the *VBoxHfs-64.efi* file in your EFI partition at */EFI/CLOVER/drivers64UEFI/*.  Replace that file with [*HFSPlus.efi*](https://github.com/JrCs/CloverGrowerPro/raw/master/Files/HFSPlus/X64/HFSPlus.efi) (it does not have to be renamed) and you should have some better success.

[\[Return\]](#index)

-

##Kernel Panics

This section will go over some of the common kernel panics and how to avoid them.

###AppleIntelCPUPowerManagement

This KP is caused by either locked MSR attempted access or a CPU without supported C & P-States.

To fix it temporarily, you can use *NullCPUPowerManagement.kext*.

You can alternatively enabled *AsusAICPUPM* in Clover.

For Ivy Bridge and newer CPUs, you can use Pike R. Alpha's [ssdtPRGen.sh](https://github.com/Piker-Alpha/ssdtPRGen.sh) script to generate an SSDT for your CPU.

For Sandy Bridge and older, enable Generate CStates and PStates in Clover.

[\[Return\]](#index)

###BluetoothHostControllerUARTTransport

This KP is seen on Skylake boards - and is related to the serial port.  Disable the serial port in BIOS (Peripherals -> Super IO -> Serial Port; or similar) and you should be able to boot.

If you have an Asus Z170-A board, you may also need the following in your config.plist -> ACPI -> DSDT -> Patches:

    <key>ACPI</key>
    <dict>
        <key>DSDT</key>
        <dict>
            <key>Patches</key>
            <array>
                <dict>
                    <key>Comment</key>
                    <string>Z170-A UART Patch</string>
                    <key>Find</key>
                    <data>
                    QdAFAQ==
                    </data>
                    <key>Replace</key>
                    <data>
                    AQIDBA==
                    </data>
                </dict>

[\[Return\]](#index)

-

##Audio

###Skylake

To allow many Skylake boards (if not all of them) to work with some of the more elegant audio solutions (see [AppleALC](#applealc)), you need to rename *HDAS* to *HDEF* in DSDT via the following edit to your config.plist -> ACPI -> DSDT -> Patches:

                <dict>
                    <key>Comment</key>
                    <string>Rename HDAS to HDEF for Realtek Audio</string>
                    <key>Find</key>
                    <data>
                    SERBUw==
                    </data>
                    <key>Replace</key>
                    <data>
                    SERFRg==
                    </data>
                </dict>
                
[\[Return\]](#index)

###AppleALC

[AppleALC](https://github.com/vit9696/AppleALC) is a kernel extension developed by [Vit9696](https://github.com/vit9696) that allows for dynamic patching of *AppleHDA.kext* in kext cache (non-destructive, works with SIP) and supports a [large range of codecs](https://github.com/vit9696/AppleALC/wiki/Supported-codecs).

It gets installed to your EFI partition (if using Clover) at */Volumes/EFI/EFI/CLOVER/kexts/Other/* (or the *10.xx* folders - but I prefer *Other*).  I did an audio-related writeup about it [here](https://www.reddit.com/r/hackintosh/comments/4sil5p/audio_mechanic_old_patchfix_removal_applealc/).  /u/TheRacerMaster wrote a tutorial for installation [here](https://www.reddit.com/r/hackintosh/comments/4e23w6/guide_native_audio_with_clover_applealckext/).

[\[Return\]](#index)

###HDMI Audio

Toleda is still the resident expert for HDMI audio - I don't have the capacity to figure out how he does it - but I'll link some of his repos here:

From his [HDMI Audio AppleHDA \[Guides\]](https://github.com/toleda/audio_hdmi_guides)

* [HD6000+/Desktop/BRIX/NUC](https://github.com/toleda/audio_hdmi_9series/tree/master/ssdt_hdmi-hd6000%2B)
* [HD4600+/Desktop/BRIX/NUC](https://github.com/toleda/audio_hdmi_8series/tree/master/ssdt_hdmi-hd4600%2B)
* [HD4000/Desktop/BRIX/NUC](https://github.com/toleda/audio_hdmi_hd4000/tree/master/ssdt_hdmi-hd4000)
* [HD3000/Desktop](https://github.com/toleda/audio_hdmi_hd3000/tree/master/ssdt_hdmi-hd3000)
* [AMD HDMI audio](https://github.com/toleda/audio_hdmi_amd-nvidia/tree/master/ssdt_hdmi-amd)
* [Nvidia HDMI audio](https://github.com/toleda/audio_hdmi_amd-nvidia/tree/master/ssdt_hdmi-nvidia)
* [ALC/HDEF audio](https://github.com/toleda/audio_ALCInjection/tree/master/ssdt_hdef)
* [Intel 100 Series](https://github.com/toleda/audio_hdmi_100series) (HD 515, 530, and 540 at the time of this writing)

His guides can be tough to figure out - but he knows his stuff.  Read through them carefully (IIRC, the AMD and NVIDIA guides require using [IORegistryExplorer](https://github.com/toleda/audio_ALCInjection/blob/master/IORegistryExplorer_v2.1.zip) to get the address of your card per [this guide](http://www.tonymacx86.com/threads/amd-nvidia-hdmi-audio-easy-guide.172023/)) and place the correct SSDT for your GPU in */Volumes/EFI/EFI/CLOVER/ACPI/patched/*.

[\[Return\]](#index)

-

##Hardware Exceptions

###Asus Z170-A

For all Skylake boards, you'll want to make sure that you disable your serial port via BIOS - but this board *still* seems to have some serial port issues (referenced [here](#bluetoothhostcontrolleruarttransport) as well).  You'll want to make sure you have the following in your config.plist -> ACPI -> DSDT -> Patches:

    <key>ACPI</key>
    <dict>
        <key>DSDT</key>
        <dict>
            <key>Patches</key>
            <array>
                <dict>
                    <key>Comment</key>
                    <string>Z170-A UART Patch</string>
                    <key>Find</key>
                    <data>
                    QdAFAQ==
                    </data>
                    <key>Replace</key>
                    <data>
                    AQIDBA==
                    </data>
                </dict>

[\[Return\]](#index)

-

##Tools

###Mounting the EFI Partition

To mount an EFI partition (in case you reboot, or eject one when you need it again), do the following in the Terminal:

    diskutil list

Take note of the drive number that you want, and also the partition number of the EFI partition.  It will look like *diskDsP* where *D* is the drive number and *P* is the partition number.

Then type the following in the Terminal to mount it:

    diskutil mount diskDsP

Replacing *D* and *P* with the respective drive and partition numbers.

[\[Return\]](#index)

###Repairing Permissions and Rebuilding Kext Cache

These are good maintenance tools - they ensure permissions are correct on the boot drive, and that the kext cache is not populated with old, or unused kexts.  They are all entered into the Terminal.

For repairing permissions:

    sudo /usr/libexec/repair_packages --repair --standard-pkgs --volume /

For rebuilding kext cache:

    sudo rm -r /System/Library/Caches/com.apple.kext.caches
    sudo touch /System/Library/Extensions
    sudo kextcache -update-volume /

Just ***make sure*** that you have the correct kexts in place - I've seen people mess up their builds because they swapped out correct kexts for incorrect ones, and were just coasting on the previous kext cache. When they rebuilt the kext cache, it pulled their feet out from under them and they had issues.

[\[Return\]](#index)
