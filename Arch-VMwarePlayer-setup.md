# Arch Linix VMWare Workstation Player Setup
\[Last updated 2022-02-24]

## Table of Contents
1. [Getting the Virtual Machine Running](#part-1-getting-the-virtual-machine-running)
   1. Download the Disk Image
   2. Check the Checksum
   3. Download VMWare Workstation Player
   4. VM Hardware Setup
2. [Installing Arch](#part-2-installing-arch)
   1. Boot Up
   2. Partition the Disk
   3. Format the Partitions
   4. Mount the Partitions
   5. Install the Required Packages
   6. Configuration
   7. Check if it Works
3. [Housekeeping](#part-3-housekeeping)
   1. Update and Upgrade
   2. Install Sudo
   3. Enable Wheel Sudo Access
   4. Create a User Account
   5. Delete and Lock the Root Password
4. [Optional Steps](#part-4-optional)
   - Change the Default Shell from BASH to Zsh
   - Install and Enable the XFCE Desktop Environment
   - Change the Default Prompt and Terminal Style

## Foreword
Before beginning, review and famaliarize yourself with the [default Arch installation instructions](https://wiki.archlinux.org/title/installation_guide). Please be aware that I am making this guide as a personal cheat-sheet, and that you follow it completely AT YOUR OWN RISK. I make no guarantees - this is meant solely as for convenience and should not be used without consulting the official documentation for any technology or service. If you do find any mistakes or issues, however, please let me know so I can fix them!

## Part 1: Getting the Virtual Machine Running
The first step in the process is making and configuring the actual virtual hardware for our machine. Par 1 will cover this process.

### Step 1: Download the Disk Image
Disk image files can be found at [archlinux.org/download](https://archlinux.org/download/). BitTorrent is recommended, but you can also select an appropriate mirror for a direct download, among other options.

### Step 2 (optional, recommended): Check the Checksum
The download page linked above has links to the appropriate hashes and checksums to ensure file integrity.

### Step 3: Download VMWare Workstation Player[^vmware-legal-disclaimer]
The Workstation Player download can be found at [www.vmware.com/.../workstation-player-evaluation.html](https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html). Be advised:
> VMware Workstation Player is free for personal, non-commercial use (business and nonprofit use is considered commercial use). If you would like to learn about virtual machines or use them at home, you are welcome to use VMware Workstation Player for free. Students and faculty of accredited educational institutions can use VMware Workstation Player for free if they are members of the [VMware Academic Program](https://labs.vmware.com/academic/licensing-overview).
[^vmware-legal-disclaimer]: This is not legal advice - you engage with 3rd party products at your own peril. I am not affiliated with VMWare in any way - this is a quote from the official website that is current as of 2022-02-23. Please review for yourself to ensure that this information is accurate and up to date!

### Step 4: VM Hardware Setup
1. Open VMWare Workstation Player and click 'Create New Virtual Machine.'
2. Click on 'Installer disc image file (iso)' in the options, and enter the path to the ISO file you downloaded in [**Step 1**](#step-1-download-the-disk-image), then click 'next.'
3. For the 'Guest operating system' select 'Linux,' and for the 'Version' select the appropriate version (as of this writing, select 'Other Linux 5.x kernel 64-bit'), then click 'next.'
4. Name the VM whatever you want (something descriptive, like "Arch Linux 64" - you can change this later), select where you want the VM files to go, then click 'next.'
5. Specify the disk capacity you need - in my case I will be using 20GB, but a minimum of 8GB is recommended. Specify whether or not you want the disk in one file or multiple - a single file may offer better performance in some cases, but may cost portability. Click 'next.'
6. Customize the hardware if you need to - in my case, I will be increasing the memory to 2048MB, and the core count to 2.
7. If everything is to your liking, you can hit 'finish.'

Success! The VM should now be ready to run. You can hit 'Play virtual machine' and proceed to the next part.

## Part 2: Installing Arch
We have our virtual hardware set up, now we need to install and configure Arch so that we can boot into and use it.

### Step 1: Boot Up
Click on 'Arch Linux install medium (x86_64, BIOS)' to boot into the installer. Alternatively, you can just wait and this will be done automatically. You should automaticaly be given the `root@archiso` user in the `~` (`/root`) directory.

### Step 2: Partition the Disk
Run `lsblk` to get an idea of what is happening in the system. If everything looks good, you can proceed to the partitioning steps:
1. Run `cfdisk /dev/sda` to start the partitioner on the SDA device.
2. Select the `gpt` label (for **G**UID **P**artition **T**able).
3. We first need to create a partition for the boot loader:
   1. Select 'New,' enter a size of '1M' (1 megabyte).
   2. Select 'Type,' then select 'BIOS boot' from the menu.
4. Create swap space (optional):
   Creating swap space may not be necessary for you, depending on how much RAM you allocated in [**Part 1.4.6**](#step-4-vm-hardware-setup). There are lots of opinions about this, and I urge you to research them before you make a decision. To summarize, if you have very little memory (~1GB or less), definitely add swap space (at least double your RAM, minimum 2GB). If you have more, add swap space if you can spare the storage space. Since my VM has 20GB of storage and I plan on using very little of it, I will be allocating 4GB for swapping.
   1. Move down to the 'Free space' row and select 'New.'
   2. Enter the size (4G in my case) and hit 'Type'
   3. Select 'Linux swap' from the menu
5. Allocate the rest of the disk for the Linux Filesystem:
   1. Move down to the 'Free space' row and select 'New.'
   2. The prefilled value should be the remaining disk space (in my case, 16GB), hit enter.
   3. If the type is not automatically filled as 'Linux filesystem' change it in the type menu.
   4. Ensure that there is no green 'Free Space' row.
 
The disk should be ready to partition. Read over the table to ensure everything is okay, then hit 'Write' and type `yes` to proceed. You can now quit the `cfdisk` partition tool. You can re-run `lsblk` to ensure the changes took effect.

### Step 3: Format the Partitions
1. Run `lsblk` to view the drives. Under `sda` there should be 2 partitions (3 if you have swap space). Identify the one you set aside for the linux filesystem, this will be the largest and typically last one in the list, 'sda3' if you have swap space, or 'sda2' if you don't. Remember which one it is.
2. Run `mkfs.ext4 /dev/<your filesystem partition>`, for me this is `/dev/sda3` to make an ext4 file system. If you don't have swap space, you can move on to the next step now.
3. If you have swap space, run `mkswap /dev/<your swap partition>`, for me this is `/dev/sda2`.
4. If you have swap space, run `swapon -a` to enable the swap space.

### Step 4: Mount the Partitions
Mount the filesystem partition you found in [**Step 3.1**](#step-3-format-the-partitions) to your `/mnt` directory, with `mount /dev/<your filesystem partition> /mnt`, in my case this is `mount /dev/sda3 /mnt`.

### Step 5: Install the Required Packages
To install Arch, we need the following packages:
1. [grub](https://archlinux.org/packages/core/x86_64/grub/): GNU GRand Unified Bootloader.
2. [base](https://archlinux.org/packages/core/any/base/): Minimal package set to define a basic Arch Linux installation.
3. [linux](https://archlinux.org/packages/core/x86_64/linux/): The Linux kernel and modules.
4. [linux-firmware](https://archlinux.org/packages/core/any/linux-firmware/): Firmware files for Linux.
5. [dhcpcd](https://archlinux.org/packages/core/x86_64/dhcpcd/): RFC2131 compliant DHCP client daemon.
7. Your text editor of choice (such as [vi](https://archlinux.org/packages/core/x86_64/vi/)). I will be using [nano](https://archlinux.org/packages/core/x86_64/nano/)[^vi-vs-nano].
[^vi-vs-nano]: Picking a text editor requires some soul searching. In most cases, it is highly recommended to install vi even if you prefer to use another option like nano, because many users and even some software assumes that you have it (it is *almost* guaranteed to be installed on most UNIX systems).

This can be done by running the command: `pacstrap /mnt grub base linux linux-firmware dhcpcd <your text editor(s)>`

### Step 6: Configuration
1. run `genfstab /mnt >> /mnt/etc/fstab` to generate the fstab file for the system.
2. (Optional) Run `cat /mnt/etc/fstab` to print the contents of our fstab file to ensure it is correct. If there are any errors, use the text editor we installed in [**Step 5.7**](#step-5-install-the-required-packages) to fix them.
3. Change the root directory to /mnt with `arch-chroot /mnt`.
4. Set a password for the root user with `passwd`. This doesn't have to be anything secure since we will be handling system security in an upcoming part.
5. Install the bootloader to `/dev/sda` with `grub-install /dev/sda`[^not-sda-partition].
6. Make the grub configuration file with `grub-mkconfig -o /boot/grub/grub.cfg`[^grub-mkconfig-o-option].
7. We are almost done, but we need to enable internet access:
   1. Use the system control utility to automatically start our DHCP client daemon whenever the system starts with `systemctl --now enable dhcpcd`[^systemctl-now-option].
   3. Count to 10 (using "one-one-tousand, two-one-thousand, ...").
   4. Test your internet connection with `ping github.com`. You should start recieving back data. If this happens, you can stop the process by pressing ctrl+c. If this returns an error, you can try running `systemctl status dhcpcd` to see what's happening to the service. Try waiting 10 more seconds for it to start up, then try again. If this still leads to an error, check the network adapter in the VM settings and make sure it is both connected and set to connect at power-on.
[^not-sda-partition]: Note that we are installing grub to the SDA device, **not** any individual SDA partition (eg. 'sda1' or 'sda2').
[^grub-mkconfig-o-option]: The `-o` option allows us to specify the output file.
[^systemctl-now-option]: The `--now` option both enabled and starts the service.


### Step 7: Check if it Works
Reboot to make sure everything works: `exit` then `reboot`, then log back into 'root' with the password you made previously. Try `ping github.com` to make sure the DCHP client daemon is running. Please refer to [**Step 6.7.4**](#step-6-configuration) if there are any errors. If everything works, congratulations! You've just installed Arch.

## Part 3: Housekeeping
While we may have Arch working now, it's not secure. We only have one user - root - who is allowed to do anything on the system. This poses a major security risk - what we typically want to do is create a user account that is not root, but has access to root priveleges on an as-needed basis.

### Step 1: Update and Upgrade
Run the `pacman -Syu` command to refresh the package database (`y`), upgrade the system (`u`), and sync packages (`-S`)[^when-to-update].
[^when-to-update]: It is reccomended you run this command regularly to update your system, however you **must check the official [news feed](https://archlinux.org/news/) to ensure that updating won't break any part of your system.** This process is inherent to the Arch Linux distribution.

### Step 2: Install Sudo
The '[sudo](https://archlinux.org/packages/core/x86_64/sudo/)' package is a necessity for all nontrivial linux machines. It stands for "super user do" and allows non-root users that have been given permission to operate with root privileges. Install this package with `pacman -S sudo`.

### Step 3: Enable Wheel Sudo Access
We need to edit the sudo configuration file to allow members of the 'wheel' group to use `sudo`. **Never directly edit this file!** Use the `visudo` command to edit it instead. This will enable syntax checking and error reporting before posting any changes to the sudo config file, as any errors might make the system unusable. The default editor for visudo, as the name implies, is vi. If you like using vi (and have in installed), you can simply enter `visudo` to edit the config file. 
For me, I want to use `nano` instead, so I will run `EDITOR=nano visudo` instead.

In the sudo configuration file, scroll down to the line which reads

```
# %wheel ALL=(ALL:ALL) ALL
```

and remove the pound sign and space so that it reads

```
%wheel ALL=(ALL:ALL) ALL
```

Save and exit.

### Step 4: Create a User Account
1. Add a new user to the system with `useradd -m <username>` where '<username>' is the desired username you want for the new user[^useradd-m-option].
2. Set a password for the new user with `passwd <username>`. You will be prompted for the new password - this one needs to be secure! Don't know what a secure password should look like? Many services like Google and GitHub have requirements and best practices guides on picking a good password for their services. Look them up to ensure that you are on the right path.
3. Add the new user to the wheel group with `usermod -a -G wheel <username>`, where `-a` signifies that we are appending the user to a group, `-G wheel` signifies the wheel group, and '<username>' refers to our user.
4. Log out of the root account with `exit`.
5. Log into the newly created user with the username and password we just created.
6. Try using `sudo` with a command only 'root' can use: `sudo pacman -Syu`. You will be prompted for your used password (not the root password). If pacman attempts to update the system (it should already be up to date though), then everything is working just fine, and your user account now has sudo access! If not, make sure that you have uncommented the right line in the sudo config file, and that your new user is a part of the wheel group.
[^useradd-m-option]: The `-m` option specifies that we want the utility to create a home directory for the new user.

### Step 5: Delete and Lock the Root Password
Since our new user now has access to root privileges with `sudo`, the root account is no longer needed and should generally not be used anymore. You can delete the root password with `sudo passwd -d root` and lock the password with `sudo passwd -l root`. If you ever need access to root again, you can unlock the root account with the passwd utility, then log in. **Be advised that since we deleted the old temporary password, unlocking the root account means that anyone can log into it without a password. If you need the root account unlocked, give it a secure password with `sudo passwd root` again.**
   
The virtual machine should now be ready to use!
   
## Part 4: Optional
While the system is ready to use, I usually change a few things and add a few features before using it.
   
### Change the Default Shell from BASH to [Zsh](https://wiki.archlinux.org/title/zsh)
1. Install the Zsh package: `sudo pacman -S zsh`
2. Run the `zsh` command to initiate Zshell. You will automatically be presented with a new user configuration manu - run through these options as you see fit.
3. Change the default shell to Zsh with `chsh -s /bin/zsh`[^chsh-list-shells]
4. Reboot.
5. Log into your user account and enter `echo $SHELL` to see what shell your user is running. You should see `/bin/zsh`.
[^chsh-list-shells]: You can run `chsh -l` to view a list of all installed shells.
   
Success! Zsh is now your default shell.
   
### Install and Enable the [XFCE Desktop Environment](https://xfce.org/)
XFCE is a great customizable lightweight desktop environment that is reasonably light on system resources, while being a fully-functional graphical desktop environment. Being lightweight on system resources is important, as virtual machines often run considerably less efficiently than bare-metal machines, making more expensive environments like KDE or Gnome less desireable on less powerful host computers.
1. Install the packages you need to start up XFCE: `sudo pacman -S xorg xfce4 xfce4-goodies lightdm lightdm-gtk-greeter` and go through the installation process. Feel free to use the default settings.
2. Set 'lightdm' to start automatically on boot: `sudo systemctl enable lightdm`.
3. Reboot.
   
You should now boot directly into the XFCE graphical interface!
   
### Change the Default Prompt and Terminal Style
I often feel like the default Zsh prompt and style doesn't give me enough information. Feel free to make your prompt as detailed or minimalistic as you prefer, but the provided example will change it to my personal preferences.
Using your text editor, edit the `~/.zshrc` file to change the prompt by assigning the format to the `PROMPT` variable. For formatting help [see this page](https://zsh.sourceforge.io/Doc/Release/Prompt-Expansion.html). 
```
PROMPT=$'%(?..[Error: %F{red}%?%f]\n)\n???[%n@%M: %~]\n???[%#]: '
```
This will create a prompt that looks like this:
```console
[127]

???[roux@archlinux: ~]
???[%]:
```
(Note that the first line prints the exit code if there was an error, if there is no error the line is not printed.)
   

