# macbook-pro-4-1-linux
a collection of tricks for properly running linux on early 2007 macbook pros 4,1 pre unibody. Some of these things may apply to similar models

## Aquiring and installing 340 NVIDIA drivers

It is important to note that before installing any nvidia drivers we must resolve a critical issue. Mac UEFI of this era have unique complexities that will not allow us to install working 340 drivers out of the box. Although I have seen attempts to resolve this issue through grub and uefi I have not been able to make it work. It's likely due to the different gpu/chipsets than the instructions I've found. The second best solution is to install linux in CSM mode. This is not ideal, but it does allow us to boot with the proprietary drivers!

the following references a post on the Mint forums https://forums.linuxmint.com/viewtopic.php?f=49&t=319324

### Change UEFI registers using setpci

*untested/not working*

To get this to work you need to be sending these commands through the grub interface at boot NOT in the commandline after loggin.

enable the grub boot menu by editing /etc/default/grub and commenting out `#GRUB_TIMEOUT_STYLE=hidden` and changing the value of `GRUB-TIMEOUT` to a reasonable value (reads as seconds). Run the command `sudo update-grub` and reboot when complete. When the new grub menu comes up press c to drop into the grub command line. Enter these commands seperately
```
setpci -s "00:01.0" 3e.b=a
setpci -s "01:00.0" 04.b=7
```
You can then install the Nvidia drivers through the prefered method and reboot. Follow the same steps as above when you come to the grub menu. If it boots beyond a black screen you can make those settings permanent by adding a script in the /etc/grub.d folder. You should be able to name it something like 42_nvidia-fix as long as you arnt using 42 already. The script should look like

```                                     
cat << EOF
setpci -s "00:1e.0" 3e.b=a
setpci -s "01:00.0" 04.b=7
EOF
```
Be sure to change the permissions to executable by using chmod +x <filename>.
  
Update grub using sudo update-grub and reboot.
  
You can validate that the settings changed using `setpci -s "00:1e.0" 3e.b` and `setpci -s "01:00.0" 04.b=7` from a terminal.

### CSM install

Because getting the UEFI method to work has not been successful for a lot of people, there is an alternative method. This involves installing linux using CSM or bios compatibility mode. This is not ideal because UEFI has benefits over bios and because we do not have traditional access to a UEFI bios screen. To do this we will have to intall in UEFI and then make system changes from a live USB.

Step one is to install linux normally. This will automatically setup UEFI. DO NOT INSTALL NVIDIA DRIVERS YET. After installation reboot back into your live USB environment. Open your partitioning tool of choice (Gparted if you use Ubuntu). Find your HDD often located at /dev/sda. Find the efi (esp?) and delete this partition. Create a new partition in that space and format it as cleared. Apply those changes and then add the bios_grub flag.

*DO NOT REBOOT BEFORE FOLLOWING THE NEXT STEP*

Mount your system partition at /media using something like Disks. Change the directory to /media and find your partition then check in that directory to find and copy the GUID-name directory. Run the following command replacing <GUID> with the name of your directory

`sudo grub-install --no-uefi-secure-boot --boot-directory=/media/mint/<GUID>/boot /dev/sda`

If this installs cleanly you are done. You can now install the Nvidia 340 drivers after rebooting.
  
  *note that some distros like Ubuntu 21.10 and up have started shipping BOTH bios and UEFI grub versions. This may not appear to be a problem until you upgrade your distro so make sure that when you restart you check for the `grub-efi-amd64-signed` package and remove it. I had to unintall and reinstall grub-pc upgrading from 21.10 to 22.04 and that fixed the issue.*

## Installing Drivers
  
### Ubuntu

add ppa and install drivers:
```
sudo add-apt-repository ppa:kelebek333/nvidia-legacy
sudo apt update
sudo apt install nvidia-graphics-drivers-340
```
https://launchpad.net/~kelebek333/+archive/ubuntu/nvidia-legacy

### Debian

*untested*

if using testing already skip to install otherwise:

add a file in /etc/apt/preferences with

```
Package: *
Pin: release a=unstable
Pin-Priority: 100
```

add a file in /etc/apt/sources.list.d with

`deb http://http.us.debian.org/debian/ testing non-free contrib main`

install drivers

`sudo apt install nvidia-graphics-drivers-legacy-340xx`

### Arch

stub

## Fix AHCI when booting from CSM (bios compatability) mode
  
When booting from CSM Apples firmware will automatically place drives in IDE mode instead of AHCI. This is solvable by adding commands from grub at boot. We can see this by typing lspci -nn in the terminal. The SATA controller will have [IDE mode] next to it.

We can test our solution by booting into the grub menu and pressing c. From here we can enter `setpci -d 8086:2828 90.b=40` and using `lspci` we should see that the SATA controller now says AHCI mode. I believe you can type `boot` from here to boot into your system. To make this fix permanant we can add a script to /etc/grub.d. Since we are not using the UEFI fix mentioned above you should be able to name this file something like 42_ahci-fix. Make the file executable using `chmod +x <filename>` replacing <filename> with the name you chose. Run `sudo update-grub` and when you reboot it should automatically have set the SATA controller to AHCI mode. You can check again using lspci -nn.

