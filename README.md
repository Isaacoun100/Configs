# Configs
This repository will host my config files, some important files that I need, and a readme file that willl document this new Linux journey where I will be migrating completely

First things first. I will be using [EndeavourOS]( https://endeavouros.com/ ) as my Linux distro because it's Arch based and because it provides a level of control to the user that other distros don't.

The real challenge of this journey is that I plan to work and game from this machine. For the last 6 years I've daily driven Linux for personal use and work, however, gaming, that's an entire different beast that I hope to be able to tame.

I will be listing the packages and tweaks so that if in the future for some reason I need to do this again to not stop from 0

## Timeshift
This is one of those package I cannot live without, It allows me to confidently change and tweak aspects of my system without the fear of accidently bricking my system. Quoting the [wiki](https://wiki.archlinux.org/title/Timeshift)

> [!NOTE]
> Timeshift helps create incremental snapshots of the file system at regular intervals, which can then be restored at a later date to undo all changes to the system.

It can be installed directly from the Arch repository 

``` 
sudo pacman -S timeshift
```

One important detail that I just learned while checking the wiki is that we can schedule the timeshift process to run in the background using [cron](https://wiki.archlinux.org/title/Cron#Configuration) however, we are installing the package [timeshift-systemd-timer](https://aur.archlinux.org/packages/timeshift-systemd-timer) which creates a systemd unit. Please read on how systemctl works in the [RedHat wiki](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-managing_services_with_systemd)

In this case we have a boot instance and an hourly instance so, we have to enable them and start them
```
systemctl enable timeshift-boot.timer
systemctl start timeshift-boot.service
```

## Arch chroot
This is a safety measure that comes preinstalled and configured on EndeavourOS, if you happen to have an accident where for example you loose your bootloader or for some reason lost access, and if Timeshift didn't work, then as a last measure you can use chroot. Follow the instructions in the EndeavourOS wiki [here](https://discovery.endeavouros.com/system-rescue/arch-chroot/)

## Magic SysRq Key (REISUB)
If you get a fatal error that absolutely freezes the system there is a resource that can be used to safely reboot the machine preventing any issue that may arise, this is the Magic Key, which allow us to give instructions directly to the kernel regardless of the state of the computer. Please follow the [wiki](https://forum.endeavouros.com/t/tip-enable-magic-sysrq-key-reisub/7576) instructions.
You must press 

> Alt + (Print) +
> **R** Switch to XLATE mode
> 
> **E** Send the signal SIGTERM to gracefully terminate all processes
> 
> **I** Send the signal SIGKILL to kill all processes that didn’t terminate gracefully
> 
> **S** Sync data to disk (flush)
> 
> **U** Unmount the filesystem (and remount as read-only)
> 
> **B** Reboot
>

## Secondary drives
In this machine I have 2 drives, one is a M.2 SSD and the other one is a HDD, I have Linux installed on the M.2 but I want to store games, documents, videos, etc on the other drive. The first issue I ran into is that every time that I try to access the disk it always asks for root permisions, so I cannot access it unless I put my password first, which totally kills the point of what we are trying to do. The solution to this is called fstab, which is a file located in ``` /etc/fstab ``` that tells the system how the disks should be mounted in order for the system to work, so the trick is to place the information of our external drive here so it mounts at boot, here is an example of how I mounted mine

```
# Mount for the extra disk
# <file system>      <mount point>   <type>  <options>                               <dump>  <pass>
UUID=(uuid here)    /mnt/extra      btrfs    defaults,noatime,compress=zstd,nofail    0    0
```
>[!NOTE]
> We need to create a static folder on root to make sure the directory exists before mounting the drive, this means that is not a good idea to use /tmp as it gets clean on boot, so were creating a folder in ```/mnt``` where the extra drive will always be.

The options section are instructions that we give the kernel to know how to treat the disk. First we have the defaults, which provide the users with rwx permissions and makes it accessible, second we got the noatime, which helps reduce the load, quoting the [wiki](https://wiki.archlinux.org/title/Fstab#atime_options)


> The noatime option fully disables writing file access times to the drive every time you read a file. This works well for almost all applications, except for those that need to know if a file has been read since the last time it was modified. The write time information to a file will continue to be updated anytime the file is written to with this option enabled.


Third we have compress=zstd which is a compressing system that it also helps the disk to prevent extra reads, and last nofail, which makes it so that if we have a fatal error on the disk for some reason the system will still attempt to boot.
