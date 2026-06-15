---
title: Bypassing Windows 11 Account Setup
url: https://www.bunniestudios.com/blog/2023/bypassing-windows-11-account-setup/
published: "2023-09-30T15:13:53Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=6835
---

# Bypassing Windows 11 Account Setup

I had the misfortune of setting up a Windows 11 machine and being confronted with creating a mandatory Microsoft account. I can’t concisely explain why being forced to create an account bothers me so much, but generally when a vendor tries this hard to get you to do something, it’s not for a user-friendly reason.

Anyways, after a bit of searching I found that [Rufus](https://github.com/pbatard/rufus) is able to create a Windows 11 boot image that can bypass the account setup requirement; but for various reasons I just wanted to modify the OEM configuration.

After poking through the Rufus source code for a bit, I found the pointy end of the stick, applied the patch, and it worked.

Here’s my notes on how I did it — mostly so I have it someplace where I won’t lose it, but also maybe because someone else might find it useful. NB: Microsoft seems to have been paying attention and hardening their setup process against work-arounds to account setup, so the shelf life of this post might not be so long.

Assuming you have a brand new machine with a Windows 11 OEM pre-install, and you have not yet turned it on:

1. On first boot, go to BIOS settings and turn off the TPM (and backdoors like Intel AMT, Absolute Persistence module, etc.), and allow third party OS boot. On my machine (a Lenovo laptop) this caused the screen to go black for quite a while on reboot as it undid the Bitlocker encryption on the pre-installed Windows volume. Decrypting the Windows volume is necessary for the next steps.
2. Grab an Ubuntu install image, put it on a USB drive, and boot the Ubuntu image using the “Try Ubuntu” selection.
3. Mount the C: volume (probably the biggest partition on the NVME drive). You may have to run *ntfsfix* on the volume first to make it writeable.
4. Edit the file at *…/Windows/panther/unattend.xml* and insert some XML (exact incantation shown below).
5. Unmount the volume and reboot.
6. When the first dialog box appears during setup, hit *Shift + F10* and type *OOBE\\BYPASSNRO* into the command prompt shell that appears. This will disable the internet connection requirement, and force a reboot of the machine to restart the setup process.
7. When you get to “Let’s connect you to a network” there should be an option now that says “I don’t have Internet”; click that, and the system should proceed to setup a local-only account.

During setup, I connected to the Internet using a wired Ethernet line, so I could easily cut the internet by pulling the cable out if things went wrong and I had to try again (if you do set up by wifi, it’s a bit more complicated to cut internet). In my trials I did end up connecting a couple times and allowing the system to update, and that didn’t impact my ability to pull off the procedure in the end.

The specific commands I used within Ubuntu to access the unattended installer manifest were:

```
sudo su
ntfsfix /dev/nvme0n1p3
mount /dev/nvme0n1p3 /mnt
nano /mnt/Windows/panther/unattend.xml

```

But the exact path to the Windows partition will probably be different depending on your OEM and hardware configuration. The right partition is probably the biggest partition, so you can use *fdisk* to inspect your disk and guess the exact path for your machine.

The XML I injected was this snippet:

```
<RunSynchronousCommand wcm:action="add">
<Order>1</Order>
<Path>reg add HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\OOBE /v BypassNRO /t REG_DWORD /d 1 /f</Path>
</RunSynchronousCommand>

```

Stick it in the first “settings” block, just after the “component” block. So overall, the top of the *unattended.xml* file on my machine ends up looking like this:

```
<?xml version='1.0' encoding='utf-8'?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
  <settings pass="specialize">
    <component name="Microsoft-Windows-Shell-Setup" processorArchitecture="amd64" publicKeyToken="xxxxxxxxxxxxxxxx" language="neutral" versionScope="nonSxS" xmlns:wcm="http://schemas.microsoft.com/WMIConfig/2002/State" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
      <OEMName>Lenovo</OEMName>
      <OEMInformation>
        <Logo>c:\windows\system32\oemlogo.bmp</Logo>
        <Manufacturer>Lenovo</Manufacturer>
        <HelpCustomized>true</HelpCustomized>
        <RecycleURL>https://www.lenovo.com/recycling</RecycleURL>
        <TradeInURL>https://www.lenovo.com/trade-in-program</TradeInURL>
      </OEMInformation>
    </component>
    <RunSynchronousCommand wcm:action="add">
      <Order>1</Order>
      <Path>reg add HKLM\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\OOBE /v BypassNRO /t REG_DWORD /d 1 /f</Path>
    </RunSynchronousCommand>
  </settings>
  .... more settings blocks below ....

```

It’s not exactly a fast or convenient procedure, but unfortunately the “just unplug network during setup” hack that populates the front couple pages of Google searches on the topic was patched. Anyways, I always disable a bunch of the security theater/DRM and back doors installed by OEMs (in addition to running an [overnight RAM test](https://www.memtest.org/), hence the need to allow third-party/unsigned OS boot), so this was only incrementally more effort on top of what I was already going to do.
