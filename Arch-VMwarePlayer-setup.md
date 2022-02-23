\[Last updated 2022-02-23 - review all footnotes]

# Part 1: Getting the Virtual Machine Running

## Step 1: Download the disk image
Disk image files can be found at [archlinux.org/download](https://archlinux.org/download/). BitTorrent is reccomended, but you can also select an appropriate mirror for a direct download, among other options.

## Step 2 (optional, reccomended): Check the checksum
The download page linked above has links to the appropriate hashes and checksums to ensure file integrity.

## Step 3: Download VMWare Workstation Player[^vmware-legal-disclaimer]
The Workstation Player download can be found at [www.vmware.com/.../workstation-player-evaluation.html](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html). Be advised:
> VMware Workstation Player is free for personal, non-commercial use (business and nonprofit use is considered commercial use). If you would like to learn about virtual machines or use them at home, you are welcome to use VMware Workstation Player for free. Students and faculty of accredited educational institutions can use VMware Workstation Player for free if they are members of the [VMware Academic Program](https://labs.vmware.com/academic/licensing-overview).

## Step 4: VM Hardware Setup
1. Open VMWare Workstation Player and click 'Create New Virtual Machine.'
2. Click on 'Installer disc image file (iso)' in the options, and enter the path to the ISO file you downloaded in [**Step 1: Download the disk image**](#step-1-download-the-disk-image), then click 'next.'
3. For the 'Guest operating system' select 'Linux,' and for the 'Version' select the appropriate version (as of this writing, select 'Other Linux 5.x kernel 64-bit'), then click 'next.'
4. Name the VM whatever you want (something descriptive, like "Arch Linux 64"), select where you want the VM files to go, then click 'next.'
5. Specify the disk capacity you need - in my case I will be using 20GB, but a minimum of 8GB is reccomended. Specify whether or not you want the disk in one file or multiple - a single file may offer better performance in some cases, but may cost portability. Click 'next.'
6. Customize the hardware if you need to - in my case, I will be increasing the memory to 2048MB, and the core count to 2.
7. If everything is to your liking, you can hit 'finish.'

Success! The VM should now be ready to run. You can hit 'Play virtual machine' and proceed to the next part.


[^vmware-legal-disclaimer]: This is not legal advice - you engage with 3rd party products at your own peril. I am not affiliated with VMWare in any way - this is a quote from the official website that is current as of 2022-02-23. Please review for yourself to ensure that this information is accurate and up to date - if it is not, please let me know!
