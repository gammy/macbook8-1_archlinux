Archlinux on the Macbook Pro 8,1 (2011)
=======================================
Arch linux configuration and information for the macbook pro8,1 (2011). Written in the hope that it might help someone else trying to run linux on the macbook, and to help me remember.
Resources
---------
* Partitioning / Pre-install
 - https://wiki.archlinux.org/index.php/MacBookPro8,1/8,2/8,3_%282011%29
* Screen / Keyboard brightness
 - https://bbs.archlinux.org/viewtopic.php?id=105091
* Touchpad
 - http://community.linuxmint.com/tutorial/view/1361
 - https://web.archive.org/web/20150210065444/http://uselessuseofcat.com/?p=74
 - https://github.com/BlueDragonX/xf86-input-mtrack

* Uncategorized:
 - https://wiki.archlinux.org/index.php/MacBook
 - https://wiki.archlinux.org/index.php/MacBook#OS_X_with_Arch_Linux
 - https://wiki.archlinux.org/index.php/REFInd
 - https://bbs.archlinux.org/viewtopic.php?id=70170
 - https://wiki.archlinux.org/index.php/general_recommendations
 - https://wiki.archlinux.org/index.php/Broadcom_wireless#b43
 - https://wiki.archlinux.org/index.php/display_manager#Graphical
 - https://wiki.archlinux.org/index.php/LightDM
 - https://wiki.archlinux.org/index.php/UEFI

Setup Goals
-----------
- Dual-boot the existing OS X installation alongside arch linux
- Install arch via Live USB stick and internet (wired; wireless might be problematic, see below)
- For linux to run better than OS X itself

Gotchas
-------
- I won't be using a fully integrated desktop environment, but if you do, some things might be easier
- With the base configuration described here, your touchpad will click on tap, right-click on triple-finger-tap, and paste on double-finger-tap
- I won't delve *too* deep into making the environment look pretty here. Stuff like making the terminal colors nicer, mouse cursor, etc is left out for the mostpart
- Some of my configuration can be found in `config/`, but remember that these might have become incompatible/outdated when you read this
- I install pulseaudio but don't show how I start it. This is just in my window manager:s startup script. The same goes for devmon and so on

Functionality
=============

Working
-------
* Video
* Audio (Builtin speakers, audio jack analog)
* Webcam
* Ethernet
* Multitouch (using `mtrack`)
* WiFi
* Software Suspend
* Display brightness control
* Multimedia keys: the following work/identify correctly out of the box: `XF86MonBrightnessDown` `XF86MonBrightnessUp` `XF86LaunchA` `XF86LaunchB` `XF86KbdBrightnessUp` `XF86KbdBrightnessDown` `XF86AudioPrev` `XF86AudioPlay` `XF86AudioNext` `XF86AudioMute` `XF86Eject`. I bound some of these keys to scripts I've written. See http://github.com/gammy/misc-scripts.git

Work in progress
----------------
- The volume up / down buttons, which ought to trigger `XF86AudioLowerVolume` and `XF86AudioRaiseVolume`, don't even trigger scan/keycodes (`xev`), so I can't use them at all at the moment.
- An unresolved problem is how to utilize the keyboard LED brightness keys. The only way I can currently configure the brightness is by writing to `/sys/`, which requires root priveleges. The only way I can think of is to handle the key events via the acpi handler (ie `/etc/acpi/handler.sh`), but these keys don't trigger acpi events!
- Touchpad / Multitouch: It's currently quite twitchy. I haven't figured out a scheme for emulating the standard X resize method (CTRL+right-drag), since there is no right button. 

Untested
--------
- Bluetooth
- SPDIF audio 

My setup process, more or less
==============================

Pre-Install
-----------
* Prepare the USB Stick. I already had one lying around, but it's simple enough using `dd`.
  - See https://www.archlinux.org/download/ and https://wiki.archlinux.org/index.php/USB_flash_installation_media
* Boot into OS X
* Launch the disk utility
* Shrink the OS X (HFS+)-partition to an appropriate size; from 500 to 140G in my case
* Turn the computer off
* Insert the USB-stick, and then turn the mac on whilst holding down the ALT-key;
  - Once the disk boot menu pops up, select the USB stick and continue.
  - Once you've booted, you're dropped into a shell.

Install
-------
* The normal installation procedure follows, except that we don't configure any bootstrapping at all(no GRUB, etc)
* Use `gdisk` to add your partitions. I wanted a simple scheme with a root, swap and home partition.
* Exit gdisk and perform your formatting / swap magic. For me, using btrfs and a swap, the end result:
<pre><code>
[gammy@lucia ~]$ lsblk -f
NAME   FSTYPE  LABEL       UUID                                 MOUNTPOINT
sda                                                             
├─sda1 vfat    EFI         70D6-1701                            
├─sda2 hfsplus Untitled    18b62878-6a7b-3fa8-bbf7-d56a42fff1cf 
├─sda3 hfsplus Recovery HD c7034e4c-d6e8-3dfc-af2b-008be1735b33 
├─sda4 swap                62dcdd3f-f6ee-4a00-9475-9c962cf602d6 [SWAP]
├─sda5 btrfs               09a733ec-3894-4b8b-b32f-7d63f2a0dd82 /
└─sda6 btrfs               42a1f9fc-1c1b-4781-91f0-decec3ed51ce /home
</code></pre>
* Continue the install as usual (`loadkeys`, `mount`, `swapon`, `pacstrap`, `arch-chroot`, `mkinicpio`, etc)

Post-Install
------------
* Reboot into OSX
* Download rEFInd and install it on your pre-existing ESP (EFI Service Partition): http://www.rodsbooks.com/refind/installing.html#osx
  This involves mounting your ESP, making a few paths, and copying a few files. Alternatively an autoinstaller exists on the same site, but I didn't use it.
* Reboot into arch linux (rEFInd will present arch as an option now)
* Temporarily connect an ethernet cable for a wired network connection
* Start it up 
 *`systemctl start dhcpcd` (note no enabling)
* Update the package database (to get updated paths)
 * `pacman -Sy`
* Install xorg basics
 * `pacman -S` `xorg-server` `xorg-server-utils` `xorg-apps` `xorg-xinit` `xorg-xev` `extra/xf86-video-intel` `extra/xf86-input-synaptics` `ttf-dejavu`
 * Note that we are not using `xf86-input-synaptics` later on, since we replace it with the unofficial `xf86-input-mtrack` later, to utilize more features of the trackpad
* Install a grapical login manager, and enable it (but don't start it)
 * `pacman -S` `lightdm` `lightdm-gtk3-greeter` 
 * Edit `/etc/lightdm/lightdm.conf` so that `greeter-session` under `SeatDefaults` is set to `lightdm-gtk-greeter` (yes, *not* gtk3, but gtk)
 * `systemctl enable lightdm`
* Install the wicd network connection manager
 * `pacman -S` `wicd` 
* Enable it (but don't start it)
 - `systemctl enable wicd`
* Install pulseaudio...
 * `pacman -S` `pulseaudio` `pulseaudio-alsa` `pavucontrol` 
* And some miscellaneous stuff to ones own taste
 * `pacman -S` `pekwm` `gvim` `devmon` `pmutils` 
* Install yaourt, an AUR (Archlinux User Repository) package manager
 * https://archlinux.fr/yaourt-en#get_it
* Install proprietary wifi drivers via yaourt
 - `yaourt -S` `core/b43-fwcutter` `core/b43-firmware`
* Install the unofficial mtrack trackpad driver
 - `yaourt -S` `xf86-input-mtrack-git`
* Create a nonroot user
* `reboot`
* Login as your nonroot user, hopefully straight into pekwm
* Configure your network from `wicd-gtk`
* Unmute audio from `pavucontrol`
* Install some post-fact configuration files, for keyboard and, more importantly, the mtrack touchpad driver 
 * `cp -r <this git repository>/root/ /`
 * This is of course optional but you should at least look in the `xorg.d/`..
* `reboot`
