# Pre-Install
## Set temporary keymap for keyboard

    loadkeys la-latin1
    

## Check network configuration

    ping -c 3 google.com
    # If necessary, fix connection issues by running: dhcpcd

## Disk partitioning
### Check which drive is which

    fdisk -l

### Create the partition table

    cfdisk /dev/sda (filesystem + swap)

### Format file system drive & mount

    mkfs.ext4 /dev/sda1 (filesystem)
    mount /dev/sda1 /mnt

### Format swap drive and set as swap

    mkswap /dev/sda2
    swapon /dev/sda2

# Instalation
## Install arch itself

    pacstrap /mnt base base-devel

## Create fstab file

    genfstab /mnt >> /mnt/etc/fstab # dotfiles for reference

## Switch into arch

    arch-chroot /mnt

## Build the linux kernell (not needed)

    mkinitcpio -p linux

## Configure users
    passwd root # sets root password
    useradd -m -g users -s /bin/bash nuevoUsuario (m crear home dir, g agregar al grupo de usuarios) # creates a user
    passwd nuevoUsuario # sets new user password

## Install bootloader in order to be able to start up

    pacman -S grub-bios os-prober
    # mkdir -p /media/OtherDrive && mount /dev/sdb1 /media/OtherDrive # mount drive with another OS for dual booting
    os-prober (should detect other operating systems)
    grub-install /dev/sda
    grub-mkconfig -o /boot/grub/grub.cfg
    
## Reboot the system
    exit
    umount /mnt
    reboot

# Post-Installation: Basic Config
## Set host name for the computer

    echo whatever > /etc/hostname

## Set persistent network configuration

    systemctl enable dhcpcd

## Refresh mirrors
In order to have faster download speeds, refresh the mirrors, rank and sort them:

    sudo pacman -S reflector --needed --noconfirm # Download reflector, a utility to rank and sort mirrors
    sudo reflector --latest 200 --protocol https --sort rate --verbose --save /etc/pacman.d/mirrorlist

## Configure sudo access
### Install sudo itself

    pacman -S sudo

### Make the wheel group sudoers

    nano /etc/sudoers # see dotfiles for reference
**Uncomment `%wheel ALL=(ALL) ALL`**

### Grant sudo access
    gpasswd -a nuevoUsuario wheel # add desired users to wheel

## Set a persistent locale (system language)

    nano /etc/locale.gen # Already included in dotfiles
    locale-gen

## Set a persistent keyboard map layout

    echo "KEYMAP=la-latin1" > /etc/vconsole.conf # not latam
    sudo nano /etc/X11/xorg.conf.d/00-keyboard.conf

    Section "InputClass"
       Identifier "system-keyboard"
       MatchIsKeyboard "on"
       Option "XkbLayout"  "latam"
       Option "XkbModel"   "pc104"
       Option "XkbVariant" "deadtilde,dvorak"
       Option "XkbOptions" "grp:alt_shift_toggle"
    EndSection


## Set a persistent time configuration
    rm /etc/localtime
    ln -s /usr/share/zoneInfo/America... /etc/localtime
    sudo pacman -S ntp
    sudo ntpd -qg

Reference:
- https://wiki.archlinux.org/index.php/time
- https://www.techgainer.com/fix-windows-showing-wrong-time-in-linux-windows-dual-boot-system/
- https://bbs.archlinux.org/viewtopic.php?id=167407

## Configure Pacman
### Include 32 bit packages

    nano /etc/pacman.conf # uncomment multilib

    [multilib]
    Include = /etc/pacman.d/mirrorlist
    SigLevel = PackageRequired TrustedOnly

### Add an AUR helper
#### Pacaur (depreciated)
##### Edit package manager file

    nano /etc/pacman.conf # add fr packages source which includes yaourt

    [archlinuxfr]
    SigLevel = Never
    Server = http://repo.archlinux.fr/$arch

##### Sync changes
    
    sudo pacman -Sy

##### Install and sync yaourt itself

    sudo pacman -Syu pacaur
    pacaur -Syu

#### Yay

    sudo pacman -S go --needed --noconfirm
    git clone https://aur.archlinux.org/yay.git ~/yay
    cd ~/yay
    makepkg -si


## Set numlock on boot
There are two components to having the numpad activated on boot. The first is the console activation which can be done by:

    sudo systemctl edit getty\@.service

    [Service]
    ExecStartPre=/bin/sh -c 'setleds +num < /dev/%I'

And the second is the display manager's activation. In case of X.org, numpadx can be installed and executed on ~/.xinitrc so that i$

    sudo pacman -S numlockx
    vim ~./.xinitrc

    numlockx &

    exec window_manager

Reference:
 - https://wiki.archlinux.org/index.php/Activating_Numlock_on_Bootup

 ## Other Stuff
sudo pacman -S ntp dbus avahi cups
systemctl enable ntpd
systemctl enable avahi-daemon
systemctl enable org.cups.cupsd.service

## Configure automatic network time update service
nano /etc/ntp.conf
??
hwclock

# Post-Installation: Sound & Graphics
## Audio setup

    sudo pacman -S alsa-firmware alsa-lib alsa-plugins alsa-tools alsa-utils pulseaudio-alsa lib32-alsa-lib lib32-alsa-plugins
    sudo pacman -S pavucontrol
    # restart?
    pavucontrol # configure

## Xorg Setup

    pacman -S xorg-server xorg-xinit xorg-apps # (xorg-apps replaces xorg-server-utils and xorg-utils)
    # not necessary:
    # pacman -S xorg-twm xorg-xclock xterm
    # pacman xorg-drivers

### i3WM
#### Install i3 itself

    yaourt -S i3-gaps
    # sudo pacman -S i3

#### Install basic i3 utils
    pacman -S dmenu i3status i3blocks

#### Configure Xorg to start i3
    su someUser
    echo "exec i3" > ~/.xinitrc

#### Start i3
    startx
    # configure i3 to open a different terminal

### KDE
#### Install KDE itself:

    pacman -S plasma-desktop
    # pacman -S sddm ?

#### Configure Xorg to start KDE

    su someUser
    echo "exec startkde" > ~/.xinitrc

# Post-Installation: Apply dotfiles
## Apply dotfiles

    sudo pacman -S git python --needed --noconfirm
    git clone https://github.com/0x6C38/dotfiles.git ~/dotfiles
    cd ~/dotfiles
    ./arch-install
    sudo ./arch-install

# Sources
- https://wiki.archlinux.org/index.php/installation_guide

- https://www.youtube.com/watch?v=lizdpoZj_vU
- https://www.youtube.com/watch?v=GCUmGtCYPWM

- https://www.youtube.com/watch?v=Wqh9AQt3nho
- https://www.youtube.com/watch?v=P4IV5BYPiPs
- https://www.youtube.com/watch?v=5Idf4lFsmx0


# Messy notes
----
timedatectl set-timezone America/Argentina/Buenos_Aires (setea la timezone correcta)
timedatectl set-ntp true
timedatectl status (muestra el estado)

hwclock --systohc --utc (sincroniza el reloj)

grub-install --target=i386-pc --recheck /dev/sda (instala el bootloader)
cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
grub-mkconfig -o /boot/grub/grub.cfg

--Post-install
localectl set-locale LANG="en_GB.UTF-8" (setea el locale de nuevo para algunas aplicaciones que no funcionan correctamente)

pacman -S xf86-input-libinput xorg-server xorg-xinit xorg-apps mesa (paquetes para instalar una interface grafica, puede haber errores de mirros, simplemente esperar hasta que encuentre alguno que funcione)

pacman -S -xf86-video-amdgpu (driver amd, sin el primer -?) tambien (generico) -S xf86-video-vesa
pacman -S gdm (display manager para gnome)
pacman -S gnome gnome-extra (paquetes de paquetes de gnome, necesarios salvo el extra, pero recomendado igualmente)
systemctl enable gdm (hace que el servicio grafico empiece automaticamente)

Missing:
- Make ntp execute as a service
- OS prober?
- Mount other drives
