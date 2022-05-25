# Instructions

These instructions were originally created to support the workshop *Practically Protecting Phone Privacy*, which is going to be presented on Sunday August 8, 2021 at the [DEFCON 29](https://defcon.org/) [Crypto and Privacy Village](https://cryptovillage.org).

## Prerequisites BEGIN
The following steps should be reviewed and performed **before** the workshop, to ensure you will get the most out of it.

### Hardware
1. A phone supported by the latest release of [LineageOS](https://lineageos.org/) - which is currently 18.1 (Android 11). To identify which phones fit this requirement, go to the LineageOS list of [supported devices](https://wiki.lineageos.org/devices/) and then disable the "Show discontinued devices" option.
1. A phone that does **not** have a **carrier locked** bootloader.
    - Stay away from carrier-exclusive phones! (Verizon, AT&T, Tracfon, etc.)
    - This is different from a device that is [SIM locked](https://en.wikipedia.org/wiki/SIM_lock).
    - SEE ALSO
        - [Can I unlock a carrier locked bootloader (T-Mobile) by using the pixel factory flash](https://forum.xda-developers.com/t/can-i-unlock-a-carrier-locked-bootloader-t-mobile-by-using-the-pixel-factory-flash.4043477/), which also talks about the differences between T-Mobile and Verizon regarding bootloader locks.
        - [SIM lock vs booloader lock](https://android.stackexchange.com/questions/44782/unlocked-device-vs-unlocked-bootloader)
1. **Phone:** We picked two phones representing higher and lower end. The fact that both are Motorola brand is completely coincidental; we first had the G7 and then looked for the cheapest phone we could get that was supported by LineageOS 18.1.
    - [**Motorola Moto G7 (River)**](https://wiki.lineageos.org/devices/river/) - This is a mid-tier 2019 phone with some fancy features, that can still be found new in box.
    - [**Motorola Nexus 6 (Shamu)**](https://wiki.lineageos.org/devices/shamu/) - This is a 2014 phone that was reasonably inexpensive in the used market (carrier/SIM unlocked on eBay for $60).
1. Appropriate USB data cable
1. Micro-SD card or 2GB+ USB storage with appropriate USB connector/adapter
1. If you plan to use this phone as a ..phone.. make sure you have a working SIM

### Computer
* You will need a computer, computer-looking thing, or virtual machine (VM). We recommend using a VM, to avoid making changes to your host OS.
* We will use this VM to unlock the bootloader and install LineageOS on the phone, and maybe a few other things.
* We are using Xubuntu - a "lightweight" Ubuntu Linux based desktop OS - running under VirtualBox. VirtualBox runs on Windows, Mac, and Linux. Details on how to install VirtualBox are listed [below](#Setup).

### VPN Server or Service
There are two options to accomplish this. Ranking them from easiest to hardest we have
1. If using a VPN service, sign up for an account
    * Examples we use:
        * ProtonVPN
            * Requires ProtonMail account
        * Calyx VPN
            * No account required
1. If using a self-hosted VPN, you will need a (virtual) computer which you can access from the Internet. If you don't already have such a computer, you can start that process by signing up for a free/paid account at your host of choice:
    * Examples:
        * AWS
        * DigitalOcean
        * Linode

### Software
You need to have the following installed 
1. (Optional but recommended) VirtualBox
1. Xubuntu

and inside the VM,
1. Android Tools
1. LineageOS for microG
1. TWRP recovery

In the following setups, we will call out specific software versions. Those are the ones we have used, but it's probably safe to assume "*the latest/most current release*."

#### Setup
Attendees should ensure they have completed the following prior to the workshop. There will not be time allocated for downloading/configuring these items.

1. Install a current version Xubuntu (baremetal, or virtual machine... or container, if you wish!). We prefer the long term (LTS) releases. As of this writing, 20.04.2.0 is the latest LTS.
    - [Download Xubuntu 20.04.2.0 ISO](http://cdimage.ubuntu.com/xubuntu/releases/20.04/release/)
    - [Create a bootable USB](https://ubuntu.com/download/iot/installation-media) and Install Xubuntu on baremetal  
    **or**
    - [Install Xubuntu in VirtualBox](http://www.fixedbyvonnie.com/2015/07/how-to-setup-xubuntu-linux-in-virtualbox-step-by-step/) (as a virtual machine)
        - [Set up a USB Filter to attach your phone to the VM](https://www.virtualbox.org/manual/UserManual.html#settings-usb)
            - You can set up a ["wildcard" filter](virtualbox-wildcard-usbfilter.jpg)  
              **Note:** this will attach any newly-connected USB devices to the VM
1. Obtain Android Tools  
    ```bash
    $ sudo apt install android-tools-adb android-tools-fastboot
    ```
1. Verify ability to interact with the phone over USB
    1. Ensure that you are using a data USB cable. Some cables are only good to provide power; those are great at DEFCON, but not so useful here.
    1. Find the phone in the USB chain.
        1. Run `lsusb` **before** attaching the phone to your Xubuntu computer.
            ```bash
            $ lsusb
            Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
            Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
            Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
            Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
            Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
            Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
            ```
        1. Plug the phone into one of your computer's USB ports.  
           Note: If your "computer thing" is a VirtualBox VM, and the chosen port is USB 3.0, you will need to enable the "USB 3.0 (xHCI Controller)" in the VM's USB settings.
        3. Run `lsusb` again. If you are using a data USB cable and the phone is turned on, it should now show up as a new entry in the USB chain. In this example, the phone is shown as "Google Inc. Nexus Device", since it is a Nexus 6.
             ```bash
             $ lsusb
             Bus 002 Device 012: ID 18d1:4ee1 Google Inc. Nexus Device (MTP)  <----------[PHONE]
             Bus 002 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
             Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
             Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
             Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
             Bus 001 Device 002: ID 8087:0024 Intel Corp. Integrated Rate Matching Hub
             Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
            ```
    1. Confirm ability to interact with the phone using `fastboot`
        1. Power on the phone in Bootloader mode
            * With the phone powered off, hold \[Volume Down\] + \[Power\] until you see a screen with an "opened" Android robot
        3. Connect the phone to your computer (or VM) via USB
        4. On your "computer thing", open a terminal and:
            1. List connected devices
                ```bash
                $ fastboot devices
                [DEVICE_SERIAL_NUMBER]	fastboot 
                ```
        1. Unplug the phone from USB
        1. On the phone, tap \[Volume Up\] or \[Volume Down\] until you see "Power Off", then press \[Power\] to turn the phone off.
1. Download LineageOS 18.1 for microG for your phone
    - [Motorola Moto G7 (River)](https://download.lineage.microg.org/river/) (20210704)
    - [Motorola Nexus 6 (Shamu)](https://download.lineage.microg.org/shamu/) (20210704)
1. Download Team Win Recovery (TWRP) for your phone
    - [Motorola Moto G7 (River)](https://dl.twrp.me/river/) (3.5.2_10-0)
    - [Motorola Nexus 6 (Shamu)](https://dl.twrp.me/shamu/) (3.5.2_9-0)
1. Download the copy-partitions package (for "A/B slot" devices)
    - [copy-partitions-20200903_1329.zip](https://androidfilehost.com/?fid=8889791610682929240)

## Prerequisites END
If you made it this far, and still have all your fingers, congratulations! You are well-prepared for the workshop.
