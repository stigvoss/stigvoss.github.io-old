---
layout: post
title: Inaccessible Boot Device caused by Hyper-V on X370 Taichi
tags: [hyper-v, windows, bluescreen, amd, ryzen, x370, asrock, bios]
categories: [Hyper-V]
image: '/images/posts/bluescreen.jpg'
---

During a session of crash diagnostics on my Windows hypervisor running  Hyper-V, I caused the operating system to become completely  inaccessible. This is not the issue referenced in the title, but the  cause for me ending up having the issue.

To resolve the previous  issue, I attempted to update the BIOS from 3.20 to 3.30 and finally to  4.60, but no luck. Defeated, I turned to reinstalling Windows.

Following the reinstall, Windows booted just fine. Though, it was discovered that when installing the Hyper-V feature, Windows would not boot anymore.

Booting with Hyper-V installed caused a INACCESSIBLE_BOOT_DEVICE bluescreen.

The hypervisor is configured as following:

- AMD Ryzen 1700
- 64 GB DDR4 RAM
- [ASRock X370 Taichi](https://www.asrock.com/mb/AMD/X370 Taichi/)
- 2x SATA SSD in RAID1 using AMD RAID for Windows

I tried experimenting with BIOS settings, disabling features such as SR-IOV based on [this thread](https://www.reddit.com/r/sysadmin/comments/5yfvzf/adding_hyperv_role_to_server_2016_stops_it_booting/), but no luck.

## Solution

Feeling out of options, I attempted to revert the changes made, starting with  the BIOS. I downgraded directly from version 4.60 to 3.20 of the BIOS.

The BIOS downgrade did it. The system booted with Hyper-V enabled following the BIOS downgrade.

I am unsure why this worked and why the BIOS upgrade affected  virtualization, but having Google'd for several hours without finding  the solution, I thought I would post it here in the hope that my  discovery could help someone else who encountered the same issue.