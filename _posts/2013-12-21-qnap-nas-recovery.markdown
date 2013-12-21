---
layout: post
title: QNAP NAS (TS-410) Receovery
tags: qnap nas ts-410 410 firmware recovery dom not staring qts4 truble shooting timemachine tm apple raid
---

My mother has a QNAP NAS TS-410 at home. Up until recently everything was working fine. She called me and said, that her Apple MacBook is not making backups anymore. I visited her and noticed that the QNAP was turned off, so backups obviously cann't be made. I thought strange, why is it offline? Nobody turned it off!

After starting the QNAP it was flashing in a red/green cycle infinitly. I waited about 10 minutes and stopped the QNAP. Not knowing what to to, I took it home for further investigation. There was no network connection to or from the QNAP, so I couldn't tell exactly what the problem was (Qfinder, my router and the network tools i know also didn't find it).

Since this is an older model I had to search a bit for the manual. But using the archive section, the correct bay count and model took me to the [right support page](http://www.qnap.com/v3/en/product_x_down/product_down.php?cat=4&type=4&II=26).

I found the [Turbo NAS Troubleshooting Guide (English)](http://eu1.qnap.com/Storage/TechnicalDocument/QNAP_Turbo_NAS_Troubleshooting_Guide_ENG.zip) in the section 'Technical Document' and read through it. I followed the different ways of troubleshooting in the guide:

1. Reboot (shutdown and start again)
2. Reset (reset network settings)
3. Without disks (shutdown, remove all disks and restart)

The only option that let the QNAP start, was the 3. one. The strange thing is, that the disks seem to be okay, at least they where green not red while starting. Now that I was able to see it using the Qfinder i thought maybe the firmware/OS of the QNAP is broken.

The manual has a section regarding "Firmware Recovery" (chapter 8). To be sure, that the firmware is working properly, I started the firmware recovery. As a precondition you have to be directly connected to the QNAP. Since I have a TS-410, I downloaded [Flash_Reburn_live-cd-2009-12-09 for TS-410](http://us1.qnap.com/Storage/tsd/Flash_Reburn_live-cd-2009-12-09(TS-410&410U%20&419P&419U).iso) links where placed in the manual page. There are two options burning a CD or creating a bootable USB [as described here](http://evan.borgstrom.ca/post/1314205955/osx-bootable-usb-from-iso). After booting that live cd you need to start the QNAP while having the power and reset button pressed. I think some network boot is happening then, you have to wait around 5 minutes until another beep tone. I tell you I have heared a lot of beeep, beeeeep, beep that day.

Using the Qfinder I was able to find the QNAP again. Still having all disks unmounted I used an old disk with no important data to complete the initial setup. The initial setup allows you, to go straigt to the current firemware/OS, which is what I did (Using the current QTS 4.0.5).

After the installation I could tell that everything was fine again, and the QNAP worked normal. Since the NAS has 4 disks and 3 slots where free I turned it off, mounted 3 of the 4 disks that I have in a RAID 5 setup and started the NAS to see if they are blocking booting the system. They didn't, looking at the `dmesg` output (login using ssh and type dmesg) was reporting, that the RAID driver expected the 4 disk. So I turned it off and added the last disk back to the array. Starting up took a bit longer this time, i noticed some disk activity after the system started (no red/green cycle) and I let the NAS on it's own all the night.

Next morning, no further action done, everything was back to normal. No data lost, current QTS (4.0.5) and all the services where operational.

I am not totaly sure, what the root cause of all this was, but knowing how to to a success desaster recovery on the QNAP, gives confidence. Maybe that helped you also.

PS: After that I installed the AfpTmFix (Version 1.1.2) to fix the time machine for apple.

Side story:

The RAID was not always operational 3 months after the buy one of the 4 disks of the RAID was broken. No problem for the QNAP and it's RAID 5. Even that I removed the disk and replaced it by the new one (since the model supports hot swap, this can be done without downtime).
