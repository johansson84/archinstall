# Installing Arch
Installation flow of Arch linux and I3 window manager 
* Set the keyboard layout

		loadkeys sv-latin1
* Update the system clock

		timedatectl set-ntp true
* Encrypt disk (LVM on LUKS)
	* Partitioning
	
			fdisk /dev/sda
		* create an empty MBR partition
		
				(fdisk) o
		* create 2 main partitions (/dev/sda1 and /dev/sda2)
		
				(fdisk) n
				(fdisk) p
				(fdisk) 1
				(fdisk) <Enter>
				(fdisk) +256M
				(fdisk) t
				(fdisk) 83
		
				(fdisk) n
				(fdisk) p
				(fdisk) 2
				(fdisk) <Enter>
				(fdisk) <Enter>
				(fdisk) t
				(fdisk) 83
		
				(fdisk) w (Write Changes)
		
		* Format Partitions
		
				mkfs.ext2 /dev/sda1
	* Setup encryption
	
			cryptsetup -c aes-xts-plain64 -y --use-random luksFormat /dev/sda2
			cryptsetup luksOpen /dev/sda2 luks
	* Create LVM partitions
	
			pvcreate /dev/mapper/luks
			vgcreate vg0 /dev/mapper/luks
			lvcreate -l +100%FREE vg0 --name root
	* Format LVM partitions
	
			mkfs.ext4 /dev/mapper/vg0-root
	* Mount system 
			
			mount /dev/mapper/vg0-root /mnt
			mkdir /mnt/boot
			mount /dev/sda1 /mnt/boot
* Install

		pacstrap /mnt base base-devel zsh openssh git gwaterfall networkmanager nm-connection-editor network-manager-applet pulseaudio mlocate vim xdg-user-dirs xterm reflector arandr htop llpp feh beaver pcmanfm catfish vlc pepper-flash flameshot
* Configure
	* Generate fstab file
	
			genfstab -U /mnt >> /mnt/etc/fstab
	* Enter new system
	
			arch-chroot /mnt /bin/bash
	* Set timezone
	
			ln -s /usr/share/zoneinfo/Europe/Stockholm /etc/localtime
	* Set locale
	
			vim /etc/locale.gen (uncomment en_US.UTF-8 and sv_SE.UTF-8)
			locale-gen
	* Edit /etc/locale.conf
	
			LANG=en_US.UTF-8
			LC_TIME=sv_SE.UTF-8
			export LANG=en_US.UTF-8
	* Set keymap
	
			echo KEYMAP=sv-latin1 > /etc/vconsole.conf
	* Set the hardware clock mode
	
			hwclock --systohc
	* Set hostname
	
			ehco hostname > /etc/hostname
		* Add it to /etc/hosts
		
				127.0.1.1	myhostname.localdomain	myhostname
	* Root password
	
			passwd
	* Create user
	
			useradd -m -g users -G wheel myusername
			passwd myusername
		* visudo
			* uncomment %wheel ALL=(ALL) ALL
	* Configure mkinitcpio with modules needed for the initrd image
		* Edit /etc/mkinitcpio.conf
			* Add 'ext4' to MODULES
			* Add 'encrypt' and 'lvm2' to HOOKS before 'filesystems'
		* mkinitcpio -p linux
	* Setup Grub
		* pacman -S grub
		* grub-install --target=i386-pc --recheck /dev/sda
		* In /etc/default/grub edit the line GRUB_CMDLINE_LINUX
			* GRUB_CMDLINE_LINUX="cryptdevice=/dev/sda2:luks:allow-discards"
		* grub-mkconfig -o /boot/grub/grub.cfg
	* Exit new system and unmount
		* exit
		* umount -R /mnt
	* Reboot
		* reboot
* Start network
	* systemctl enable dhcpcd.service
	* systemctl start dhcpcd.service
* Sort mirrors by speed
	* reflector --latest 200 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
* Set zsh as default shell
	* chsh -s $(which zsh)
	* log in and out
	* start term and config
* Setup pacman 
	* Uncomment Color line in /etc/pacman.conf
* Create user dirs
	* xdg-user-dirs-update
* Install Xorg
	* sudo pacman -S xorg
* Install graphics driver
	* Virtualbox
		* sudo pacman -S virtualbox-guest-utils
		* sudo systemctl enable vboxservice.service
		* sudo systemctl start vboxservice.service
* Install Display manager
	* sudo pacman -S lightdm lightdm-gtk-greeter
	* sudo systemctl enable lightdm.service
* Install Window manager
	* sudo pacman -S i3-wm
* Set keymap layout in Xorg
	* localectl set-x11-keymap se
* Install oh-my-zsh
	* sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
* Install network manager
	* sudo pacman -S 
	* Enable network manager
		* sudo systemctl enable NetworkManager.service
* Install yaourt
	* sudo pacman -S --needed base-devel git wget yajl
	* cd /tmp
	* git clone https://aur.archlinux.org/package-query.git
	* cd package-query/
	* makepkg -si && cd /tmp/
	* git clone https://aur.archlinux.org/yaourt.git
	* cd yaourt/
	* makepkg -si
* Install pulseaudio applet
	* yaourt pa-applet-git
* Install rofi
	* yaourt rofi
	* edit i3 config
* Install chrome
	* yaourt google-chrome
* Install archiver
	* yaourt squeeze-git
* update mlocate db
	* sudo updatedb
* Install sflock
	* yaourt sflock
* Install xfontsel
	* yaourt xfontsel

