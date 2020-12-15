## FIRST THING FIRST

This installation list is based on my preferences while trying out Debian 10 (Buster) Xfce :mouse: on my - quite old - macbook and is not a definitive procedure. Feel free to use at your own peril.
* *Note: As of mid 2020 it looks that KDE's performance is better than XFCE in certain circumstances, if this trend continues, it may be better to change to KDE as the new lightweight desktop enfironment for old computers.*

<br />

### Settings and Partitions

:computer: Apple MacBook "Core 2 Duo" 2.0 13" (Unibody) Late 2008 Aluminum AKA MacBook5,1

Intel Core2 Duo P7350 @ 2x 1.995Ghz | amd64 EFI | 4GB RAM | NVIDIA GeForce 9400M @ 256MB

| Partition | File System | Mount     | Size              | Flags    |
| ---       | ---         | ---       | ---               | ---      |
| sda1      | fat32       | /boot/efi | 142MiB            | boot,esp |
| sda3      | ext4        | /         | 141.28GiB         |          |
| sda2      | linuxswap   |           | 7.63GiB (7812MiB) | swap     |

UEFI Boot with GUID Partition Table

---

<br />

## WHILE IT'S STILL FRESH

<br />

- [x] **Update OS**
```
sudo apt update
sudo apt upgrade
```

<br />

- [x] **Install Broadcom Wireless**
* Add a "contrib" and "non-free" components to your existing repository line in `/etc/apt/sources.list` using nano, vi or other editor. Below is an example of said file in Debian 10. 
```
# See https://wiki.debian.org/SourcesList for more information.
deb http://deb.debian.org/debian buster main contrib non-free
deb-src http://deb.debian.org/debian buster main contrib non-free

deb http://deb.debian.org/debian buster-updates main contrib non-free
deb-src http://deb.debian.org/debian buster-updates main contrib non-free

deb http://security.debian.org/debian-security/ buster/updates main
deb-src http://security.debian.org/debian-security/ buster/updates main
```
* *Note:* Do not add a new line. Just add "contrib non-free" to the end of your existing line.
* Update the list of available packages:
```
sudo apt update
```
* Install the appropriate firmware installer package:
For devices with a BCM4306 revision 3, BCM4311, BCM4318, BCM4321 or BCM4322 chip, install firmware-b43-installer:
```
sudo apt install firmware-b43-installer -y
```
* Reboot.

More information can be found in [Debian Wiki](https://wiki.debian.org/bcm43xx) apple wifi webpage.

<br />

- [x] **Fixing WiFi Connection Problems After Reboot**
* If after rebooting the wireless network device can scan, but will not complete connecting, we can try turning off MAC address randomization.
* Edit `/etc/NetworkManager/NetworkManager.conf` by adding:
```
[device]
wifi.scan-rand-mac-address=no
```

<br />

- [x] **Enable Tapping and Reverse Scrolling (Natural) on Touchpad**
* Create the config file `/etc/X11/xorg.conf.d/40-libinput.conf`. We'll enable **tapping** and **reverse scrolling** by adding the following lines to said file:
```
Section "InputClass"
    Identifier "libinput touchpad catchall"
    MatchIsTouchpad "on"
    MatchDevicePath "/dev/input/event*"
    Driver "libinput"
    Option "Tapping" "on"
    Option "NaturalScrolling" "true"
EndSection
```
* Restart the DM:
```
systemctl restart lightdm
```

More information can be found in [Debian Wiki](https://wiki.debian.org/SynapticsTouchpad#Enable_tapping_on_touchpad) synaptics touchpad webpage.

<br />

- [x] **Sound (For MacBook Aluminum - late 2008)** :speaker:
```
sudo alsamixer
```
* Set all up Master, PCM, Line-Out and switch from 2ch to 6ch.
* Search for the **Volume** plug in (PulseAudio Plugin) and add it to the *Panel*, and make sure the **Enable keyboard shortcuts for volume control** is `ON`.

<br />

- [x] **Enable System Sounds**
* XFCE4 supports freedesktop system sounds, but it is not configured out of the box, therefore:
```
sudo apt install libcanberra libcanberra-pulse gnome-session-camberra
```
* Add `canberra-gtk-module` to the environment variables:
```
sudo nano /etc/environment
```
```
# For the libcanberra-gtk-module to work
export GTK_MODULES=canberra-gtk-module
```
* In *Settings Manager -> Appearance -> Settings* check **Enable event sounds**
* In *Settings Manager -> Settings Editor -> xsettings/Net/SoundThemeName* from `default` to a sound theme located in `/usr/share/sounds/`
* Make sure that **System Sounds** in the audio mixer is **turned on** and not **0**.

This process is based on the information from [ArchLinux Wiki](https://wiki.archlinux.org/index.php/Xfce#Sound_themes) webpage and [this](https://forum.xfce.org/viewtopic.php?id=8618) forum thread.
* *Note: The sound-theme-freedesktop that comes preinstalled provides a compatible sound theme, but it lacks many required events. A better choice is [sound-theme-smooth](https://aur.archlinux.org/packages/sound-theme-smooth/) (SoundThemeName should be "Smooth").*

<br />

- [x] **Dual Displays** :desktop_computer:

While XFCE4 is the lightweight linux desktop environments, it is not the friendliest when using multiple displays. This comes from the fact that XFCE treats the display arrangement as one big workspace. So, to simplify things, XFCE prefers to arrenge the 2nd monitor to the right of the primary display, which may not be the configuration that you prefer (I have my Laptop usually on the right). Also, the out-of-the-box `nouveau` drivers limits the max resolution of the external monitor if connected through VGA, thus we may want to change the Graphic Card drivers to the propietary `NVIDIA` ones.

* The NVIDIA graphics processing unit (GPU) series/codename of an installed video card can usually be identified using the `lspci` command.
```
:$ lspci -nn | egrep -i "3D|Display|VGA"
02:00.0 VGA COMPATIBLE CONTROLLER [0300]: NVIDIA Corporation C79 [GeForce 9400M] [10de:0863] (rev b1)
```
* Or to see which drivers are in use:
```
:$ lspci -k | grep -EA3 "3D|Display|VGA"
02:00.0 VGA compatible controler: NVIDIA Corporation C79 [GeForce 9400M] (rev b1)
        Subsystem: Apple Inc. MacBook5,1
        Kernel driver in use: nouveau
        Kernel modules: nouveau
```
* The `nvidia-detect` script can also be used to identify the GPU and the recommended driver package to install:
```
# apt install nvidia-detect

:$ nvidia-detect
Detected NVIDIA GPUs:
02:00.0 VGA compatible controller [0300]: NVIDIA Corporation C79 [GeForce 9400M] [10de:0863] (rev b1)

Checking card:  NVIDIA Corporation C79 [GeForce 9400M] (rev b1)
Your card is only supported upo to the 340 legacy drivers series.
It is recommenmded to install the
    nvidia-legacy-340xx-driver
package.
```
* Before installing new drivers, you must obtain the proper kernel headers for the drivers to build with.
```
# apt install linux-headers-amd64
```
* Make sure "contrib" and "non-free" components are in /etc/apt/sources.list (review)
* Update the list of available packages. Install the NVIDIA driver package:
```
# apt install nvidia-legacy-340xx-driver
```
A warning message may appear while running the Package Configuration:
```
Conflicting nouveau kernel moduoe loaded
The free nouveau kernel module is currently ooaded and conflicts with the non-free nvidia kernel module.
The easisest way to fix this is to reboot the machine once the installation has finished.
<Ok>
```
* DKMS will build the `nvidia` module for your system, via the `nvidia-legacy-340xx-kernel-dkms` package. After, create an Xorg server configuration file by installing the `nvidia-xconfig` package, then run it with `sudo`. It will automatically generate a Xorg configuration file at `/etc/X11/xorg.conf`.

* Reboot your system to enable the nouveau blacklist.
```
:$ systemctl reboot
```
* After the system reboots, connect the external monitor, open the `xfce4-display-settings` and configure your setup. (i.e., HP 23", Resolution - 1920x1080, 60.0Hz, Rotation - Left or None, position (0,0)). Make sure to set the `Laptop` display as the Primay display.

* *Note: You may want to deselect the 'Configure new displays when connected' option.*

```
So maybe search for ways to fix lightdm?
```
---

<br />

## TERMINAL STUFF

<br />

- [x] **Enable and Start Firewall**
```
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
```
* If more ports need to be open (eg. port 80 for web server), use the comand `sudo ufw allow` and then the *Port ##*. Finally, enable UFW (Uncomplicated FireWall).
```
sudo ufw enable
sudo ufw status
```

<br />

### Installing Apps

- [x] **GParted** 
```
sudo apt install gparted
```

- [x] **GIT**
```
sudo apt install git
```

- [x] **VLC**
```
sudo apt install vlc
```

- [x] ~~**Firefox**~~
```
sudo apt install firefox
```

- [x] **LaTeX**
* This will install `texlive-latex-recommended`, `texlive-fonts-recommended`, `texlive-latex-base` and `texlive-base`. 
```
sudo apt install texlive
sudo apt install latexmk
```

- [x] **Skype for Linux**
```
wget https://go.skype.com/skypeforlinux-64.deb
sudo apt install ./skypeforlinux-64.deb
```

---

<br />

## COSMETICS

<br />

- [x] **Install Themes and Icons**
* Open *Synaptic Package Manager* and search for **-theme**, **-icon** and **greeter**
* Select and install:
```
arc-theme
papirus-icon-theme
lightdm-gtk-greeter-settings
```
* Open the *Settings Manager*
* In *Appearance -> Style* select the to **Arc-Dark**
* In *Appearance -> Icons* select the to **Papirus-Dark**
* In *Window Manager -> Style* select the to **Arc-Dark**
* Also change the **Style, Icons, etc.** in the *LightDM GTL+ Greeter settings*
* Finally, consider adding the **Whisker Menu** on the *Panel Preferences -> Items*

<br />

- [x] **Advanced Themes and Icons**
* Most themes and icons can be downloaded from [Xfce-look.org](https://Xfce-look.org); Flat-Remix is a good choice.
* The basic details of the procedure are:
1. Download the archive.
2. Extract it with the right click of your mouse.
3. Create the **.icons** and **.themes** folders in your home directory. The fastest way to do that is by running in the terminal: `mkdir ~/.icons ~/.themes`.
4. Move the extracted theme folders to the ~/.theme folder and the extracted icons to the ~/.icons folder. You can make the .theme and .icons folders visible by pressing Ctrl+H or in the menu of your file manager *View -> Show Hidden Files*.
* More detailed information about Themes and Icons can be found [here](https://averagelinuxuser.com/xfce-look-modern-and-beautiful/).

* If some of the icon themes show a Xfce icons gtk-update-icon-cache warning, run the following command in the terminal:
```
gtk-update-icon-cache ~/.icons/yourIconsTheme
```

<br />

- [x] **Configure Greeter**
* Create user configuration file for **lightDM**, the greeter that asks for user and credentials, and edit it:
```
sudo nano /usr/share/lightdm/lightdm.conf.d/01_my.conf
```
* Add the following lines to the file and save :floppy_disk: :
```
[SeatDefaults]
greeter-hide-users=false
```
* After rebooting, the user name will be prompted.

<br />

### KEYBOARD SHORTCUTS

<br />

Go to *Settings Manager -> Keyboard -> Application Shortcuts*.

- [x] **Terminal**
* `xfce4-terminal` runs with *Ctrl+Alt+T*

- [x] **Dropdown Terminal**
* `xfce4-terminal --drop-down` runs with *Super+T*

- [x] **Kill Frozen Applications**
* `xkill` runs with *Ctrl+Alt+Esc*

---

<br />

## FINISHING (OPTIONAL) TOUCHES

<br />

- [x] **Atom** :link: [atom.io](https://atom.io)
* To install Atom on Debian, Ubuntu, or related distributions, add our official package repository to your system by running the following commands:
```
wget -qO - https://packagecloud.io/AtomEditor/atom/gpgkey | sudo apt-key add -
sudo sh -c 'echo "deb [arch=amd64] https://packagecloud.io/AtomEditor/atom/any/ any main" > /etc/apt/sources.list.d/atom.list'
sudo apt-get update
```
* You can now install Atom using apt-get (or apt on Ubuntu):
```
# Install Atom
sudo apt-get install atom
# Install Atom Beta
sudo apt-get install atom-beta
```
* Alternatively, you can download the Atom .deb package and install it directly:
```
# Install Atom
sudo dpkg -i atom-amd64.deb
# Install Atom's dependencies if they are missing
sudo apt-get -f install
```

<br />

- [x] **LaTeX for Atom** :link: [atom.io](https://atom.io)
* To compile latex files from within Atom, use the `latex` [package](https://atom.io/packages/latex). Just enable *Build on Save* and *Enable SyncTeX*.
* To display the generated PDF in Atom you need `pdf-view` [package](https://atom.io/packages/pdf-view). And make sure that *Auto reload on update* is enabled.
* For the Syntax highlighting and snippets, use the `language-latex` [package](https://atom.io/packages/language-latex). 
* ~~For an undistracted writing experience check out Typewriter (https://atom.io/themes/pen-paper-coffee-syntax).~~

<br />

- [x] **Jupyter Notebook** :link: [jupyter](https://jupyter.readthedocs.io/en/latest/index.html)
* First make sure you have [pip](https://jupyter.readthedocs.io/en/latest/glossary.html#term-pip) for Python installed. If not, just run `sudo apt install python3-pip` and then run the command `pip3 -V` to verify the installation.
* Then install the Jupyter Notebook using:
```
pip3 install jupyter
```
* Congratulations, you have installed Jupyter Notebook. To run the notebook:
```
jupyter notebook
```
* If you receive an error message such as `bash: jupyter: command not found` then it is possible that `~/.local/bin` is not on your path. Fix this by running `export PATH=$PATH:~/.local/bin` for your current session, then either log out and in or run `source ~/.bashrc`, then run `jupyter notebook` and it should open in your browser.

<br />

- [x] **Zotero** :link: [zotero.org](https://zotero.org/download/)
* Go to [zotero.org](https://zotero.org/download/) and `Download` the tarball file for Linux 64-bit.
* Extract the contents of the `Zotero-...x86_64.tar.bz2` tarball and move all the files into a new directory `/opt/zotero`.
* Run  the `/opt/zotero/set_launcher_icon` script from a terminal with `./set_launcher_icon` to update the .desktop file for that location.
* Symlink `zotero.desktop` for Zotero to appear in the launcher with `ln -s /opt/zotero/zotero.desktop /usr/share/applications/zotero.desktop`.
* Edit said file to point to the correct script:
```
sudo nano /usr/share/applications/zotero.desktop
```
* Edit a few lines to the following and save :floppy_disk: :
```
[Desktop Entry]
Name=Zotero
Comment=Open-source Reference Management Tool (Stand Alone)
Exec=/opt/zotero/zotero
Icon=/opt/zotero/chrome/icons/default/default256.phg
Type=Application
Terminal=false
Categories=Office;
MimeType=text/plain
```
* After rebooting, the application should run from the menu.

---

<br />

## FINAL APP CLEANING

<br />

- [x] **Uninstall Apps**
* Uninstall non needed apps using the `sudo apt purge PROGRAM` command.

- [x] **Clean Up OS**
* After uninstalling, make sure there are no dependencies left by running the following commands:
```
sudo apt autoremove -y
sudo apt autoclean -y
```

<br />
:tada: THE END :tada:

<br />
<br />

---

<br />
<br />

## PERSONAL DE SETTINGS IN XFCE

Following are my personal settings for the DE in XFCE.

<br />

- [x] **Appearance**
* *Style* -> **Flat-Remix-GTK-Blue-Dark**
* *Icons* -> **Flat-Remix-Blue-Dark**

- [x] **Window Manager**
* *Style* -> **Flat-Remix-GTK-Blue-Dark**
* *Style | Button layout* -> Remove the **Minimize** button
* *Keyboard* -> `Tile window to the top` runs with **Super+Up**
* *Keyboard* -> `Tile window to the bottom` runs with **Super+Down**
* *Keyboard* -> `Tile window to the left` runs with **Super+Left**
* *Keyboard* -> `Tile window to the right` runs with **Super+Right**

- [x] **LightDM GTK+ Greeter settings**
* *Appearance* -> Select **Arc-Dark** for the Theme
* *Appearance* -> Select **Papirus-Dark** for the Icons
* *Appearance* -> Change the Default user image

- [x] **File Manager Preferences**
* *Display* -> View new folders using: **Detailed List View**

- [x] **Workspaces**
* *General* -> Number of wokspaces: **3**

- [x] **Terminal Preferences**
* *Drop-down* -> Set Height to **70 %**
* *Appearance* -> Set **Transparent background** to **0.70** Opacity
* *Appearance* -> **Uncheck** Display menubar in new windows

- [x] **Desktop**
* *Background* -> **Uncheck** Apply to all workspaces
* *Background* -> Change the Folder: **path**
* *Icons* -> **Uncheck** Home on the Default Icons
* *Icons* -> **Uncheck** Filesystem on the Default Icons
* *Icons* -> **Uncheck** Trash on the Default Icons

- [x] **Panel**
* **Detailed List View** the 2nd Panel
* *Items* -> The Items list is as follows:
  * **Whisker Menu**
    * *Appearance* -> **Uncheck** Show application descriptions
    * *Appearance* -> **Check** Show menu hierarchy
    * *Behavior* -> **Check** Position search entry next to panel button
    * *Behavior* -> **Check** Position categories next to panel button
    * *Commands* -> Run the `xfce4-taskmanager` command for the Edit Profile
  * **Separator** -> Transparent
  * **Separator** -> Separator
  * **Show Desktop**
  * **Launcher** -> File Manager
  * **Launcher** -> Firefox ESR
  * **Trash Applet**
  * **Window Buttons**
    * *Filtering* -> **Check** Show only minimized windows
  * **Separator** -> Transparent; Expand
  * **Separator** -> Separator
  * **Power Manager Plugin** -> Show Percentage
  * **Battery Monitor** -> **Check** only Display power
  * **PulseAudio Plugin**
  * **Notification Area**
    * *Appearance* -> Maximum icon size (px): **22**
    * *Appearance* -> **Uncheck** Show frame
  * **Clock**
* *Display* -> Output: **Primary**
* *Display* -> Make the Row Size (pixels): to **28**
* *Appearance* -> Set Alpha: to **0**
* **Move** Panel 1 to the bottom

- [x] **User Icon in Whisker Menu**
* I don't like the red silouwete icon for the user on Flat-Remix, so I will change it to the Debian logo.
```
mv ~/.icons/flat-Remix-Blue-Dark/apps/scalable/system-users.svg ~/.icons/flat-Remix-Blue-Dark/apps/scalable/system-users.svg.OLD
mv ~/.debian_setup/UserIcons/debian.svg ~/.icons/flat-Remix-Blue-Dark/apps/scalable/system-users.svg
```

<br />
:octocat:
