
1) Check for internet connection:
    * `ping -c 3 archlinux.org`
    * if on wifi, use: `iwctl` to setup internet

2) System clock:
    * `timedatectl set-ntp true`

3) Partition disks following this general layout (for EFI systems):
    | Partition  |  Size        | Type                  |
    |------------|--------------|-----------------------|
    | /dev/sda1  | 600 MB       | EFI System Partition  |
    | /dev/sda2  | Entire Disk  | Linux Filesystem      |

4) Format the newly created partitions and set up swap for use:
    * `mkfs.ext4 /dev/sda2`
    * `mkfs.fat -F32 /dev/sda1`
  
5) Mount necessary partitions, create boot dir: 
    * `mount /dev/sda2 /mnt`
    * `mkdir /mnt/boot`
    * `mount /dev/sda1 /mnt/boot`

6) Install essential packages needed for minimal install (if on AMD system, exchange 'intel-ucode' for 'amd-ucode'): 
    * `pacstrap /mnt base base-devel linux linux-firmware linux-headers dhcpcd intel-ucode sudo nano htop git`

7) Generate content for fstab file at `/mnt/etc/fstab`: 
    * `genfstab -U /mnt >> /mnt/etc/fstab`

8) Change root into the new system: 
    * `arch-chroot /mnt`

9) Set time zone: 
    * `ln -sf /usr/share/zoneinfo/America/New_York /etc/localtime`

10) Set hardware clock from the system clock to update `/etc/adjtime`:  
    * `hwclock --systohc`

11) Add (or just use nano to un-comment existing) en_US to `/etc/locale.gen`
    * `echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen && locale-gen`

12) Create the file `locale.conf`, add the LANG variable to it: 
    * `echo "LANG=en_US.UTF-8" >> /etc/locale.conf`

13) Set the system hostname (eg: MYHOSTNAME): 
    * `echo "MYHOSTNAME" >> /etc/hostname`

14) Open the `/etc/hosts` file, and make it look like this: 
    ```
    127.0.0.1	localhost
    ::1		    localhost
    127.0.1.1	MYHOSTNAME
    ```

15) Set the root user's password: 
    * `passwd`

16) Create a non-root user (eg: 'USER'), and set it up:
    * `useradd MYUSER`
    * `groupadd sudo`
    * `usermod -aG sudo MYUSER`
    * `echo "MYUSER ALL=(ALL) ALL" >> /etc/sudoers`
    * `mkhomedir_helper MYUSER`
    * `passwd MYUSER`

17) Install the systemd-boot bootloader: 
   * `bootctl install`

18) Create the file `/boot/loader/entries/arch.conf`, and make it look like this: 
    ```
    title   Arch Linux
    linux   /vmlinuz-linux
    initrd  /intel-ucode.img
    initrd  /initramfs-linux.img
    options root=UUID=57fb01ee-ed7a-41c3-a18e-286842b65d94 rw
    ```
    the UUID can be added to this file more easily by running: 
    * `blkid /dev/sda2 >> /boot/loader/entries/arch.conf;`

19) Create the file `/boot/loader/loader.conf` and make it look like this: 
    ```
    default arch.conf
    timeout 3
    console-mode max
    editor no
    ```

20) Verify boot options: 
    * `bootctl list`

21) (optional) if using wifi, install iwd to use wifi after installing, enable service: 
    * `pacman -S iwd`
    * `systemctl enable iwd`

22) Start dhcpcd: 
    * `systemctl enable dhcpcd`

23) Reboot, and *hopefully* boot into a nice fresh and clean OS. Exit chroot with **CTRL-D**, and:
    * `reboot`

****** 
### **Optional post-install tasks** 

* Install the suckless DWM window manager, like the chad that you are: 
    * `sudo pacman -S xorg xorg-xinit`
    * `sudo mkdir /usr/src/suckless && cd /usr/src/suckless`
    * `sudo git clone git://git.suckless.org/dwm`
    * `sudo git clone git://git.suckless.org/st`
    * `sudo git clone git://git.suckless.org/dmenu`
    * `cd dwm && sudo make clean install`
    * `cd ../dmenu && sudo make clean install`
    * `cd ../st && sudo make clean install`
    * `echo "exec dwm;" >> ~/.xinitrc`
    * start DWM by running: `startx`
    
* Clone and install yay, an AUR helper: 
    * `cd /opt`
    * `sudo git clone https://aur.archlinux.org/yay.git`
    * `sudo chown -R MYUSER:users ./yay`
    * `cd yay && makepkg -si`

* Install some basics:
    * `sudo pacman -S firefox chromium evolution terminator neofetch`
