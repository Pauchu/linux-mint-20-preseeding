# A short rundown on how to do unattended installation in Linux Mint 20

As I was in need of finding a way to do an unattended installation for Mint 20, I found it hard to find much information on it, forcing me to piece together a solution from a lot of different sources (that I, frankly, lost track of somewhere in between). 
Be warned, however, this solution has only been tested on a handful of machines, I do not take liability for any damage caused to hardware or data.

For this solution, I am assuming you downloaded an .iso-image, already verified it and mounted it onto an USB using Rufus (though other programms should work aswell).

In order to make automated installation possible, you will need to set some commands in the preseed.cfg files first:

```
# Choose whatever locale you need
d-i debian-installer/locale string en_US.UTF-8
d-i debian-installer/language string en
d-i debian-installer/country string US

d-i netcfg/get_hostname string <Your hostname>

d-i passwd/user-fullname string <first user>
d-i passwd/username string <user>
d-i passwd/user-password-crypted password <hash>
# You can also set the password in clear text, though it is not recommended

# Choose whatever time zone you need
d-i time/zone string Europe/Berlin
# Especially important, when you have a local NTP server, and use time critical applications like Kerberos
d-i clock-setup/ntp-server string time.server.com

# This will use the self-made recipe for partitioning: 
# 100MB boot partition 
# 56GB root partition 
# 10GB tmp partition
# whatever is left of the hard drive for the home partition
d-i partman-auto/choose_recipe              select fueclient
# other predefined recipes are explained in the preseed.cfg files

# Any command you want to run after installation
ubiquity ubiquity/success_command string \
  in-target firstcommand;\
  in-target secondcommand;

# Whether you want to power off or reboot after installation
ubiquity ubiquity/reboot boolean true
#ubiquity ubiquity/poweroff boolean true
```

The reason there is two preseed.cfgs is, that, depending on if you are using an NVME or SATA controller for your harddrives, the first hard drive is called /dev/nvme0n1 or /dev/sda.
Simply using the first device is an option, but that could result in the installer trying to write onto you installation USB, frying it in the process, which we would not like.

After you setup both your preseed files, copy them into the /preseed/ directory on your of your installation media.

The isolinux.cfg is the file, that is going to be called, when you are using an old style BIOS. Just replace the isolinux.cfg found in the root of the installation media.
For an UEFI, you will need to replace the /boot/grub/grub.cfg.

I recommend not setting the boot priority for the USB over the hard drive to boot from it, as that would prompt the installation again, after the reboot. Rather you should manually select the USB as the boot device, so that the system will only boot from stick once.

As you boot from the stick, the system will show the normal grub menu, as if you did not alter the stick, but you will see two new options for unattended installation.

As I said, I only tested on a handful of machines, so no guaranties given, please run everything on a test machine, before you deploy.
