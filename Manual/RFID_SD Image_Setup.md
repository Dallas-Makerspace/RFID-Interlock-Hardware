Dallas Makerspace Legacy RFID Lockout SD Card Setup

# 1 Create New SD Card image

## 1.1 Obtain Raspberry Pi OS Image File

Download or otherwise obtain a copy of the latest Raspberry Pi OS (was called Raspbian.) Select the lightest 32-bit version available (no desktop, etc.)

[https://www.raspberrypi.com/software/](https://www.raspberrypi.com/software/)

## 1.2 Card Selection

The best option appears to be "high endurance" SD cards of 8 GB or greater.
These are usually made for security video and other high write duty cycle applications.
 
## 1.3 Raspberry Pi OS Installation

Using the SD card imager of your choice, or the Raspberry Pi foundation Raspberry Pi Imager, install an image on

You can get an image from the Raspberry Pi site that includes a Raspberry pi OS installer, but these have historically been limited to the (large) desktop version.

### 1.3.1 balenaEtcher

Available for Windows, MacOS, and Linux, this is a simple and very effective application.

https://balenaetcher.org/

### 1.3.2 Win32DiskImager

| **Caution!**** There are several sites that purport offer Win32DiskImager downloads that add malware.** |
| --- |

The correct 'official' site is:

[https://sourceforge.net/projects/win32diskimager/files/latest/download](https://sourceforge.net/projects/win32diskimager/files/latest/download)

## 1.4 Raspberry Pi OS Configuration

On first boot you will be asked for a language, username and password. Use:

- Select US English
- Username keymaster,
- Password dmsDMS

Run:

**sudo raspi-config**

- Update raspi-config tool to latest version. If you just downloaded the OS, it is likely the tool is the latest version, but this is not guaranteed.

## 1.5 Make raspi-config changes:

### 1.5.1 System Options

- 1 System|S4 Hostname Interlock-Test-1
- S5 Boot/Autologin|B2 Console Autologin
- S6 Network at boot (do not wait for network)
- S8 Power LED – change to indicates power if option is available

### 1.5.2 Display Options (none)

### 1.5.3 Interface Options

- I2 SSH (Enable)
- I4 SPI (enable) if PiFace is used (Legacy only)

### 1.5.4 Localization

- L1 Locale EN\_US.UTF-8 (disable EN\_GB if selected) and set en\_US as default
- L2 Timezone America/Chicago
- L3 Keyboard Generic 104-Key/US default layout no compose key
- L4 WLAN Country US (United States)

### 1.5.5 Advanced Options

- A4 Network Interface Names (Disable predictable network I/F Names)
- AA Network Config|dhcpcd

NOTE: If this image is intended to be copied to other cards, it's best to hold off on the following config until the final card it used.

The A1 Expand Filesystem option expands the Pi OS image to fill the entire card. This will restrict the use of the image to cards larger than the image

- A1 Expand Filesystem – (hold till complete)

Finish and reboot

## 1.6 Remove avahi

Avahi generates mDNS traffic that adds no value in this application. To remove it:

sudo apt-get purge avahi-daemon

## 1.7 Install and Configure NTP

[time details are changing.  The refernced time server is gone, and NTP may no longer nbe essential.]

We need ntp for accurate time in logs. Ntpdate is a useful command for managing and viewing ntp status.

sudo apt-get install ntp

sudo apt-get install ntpdate

edit the Timedatectl configuration file

sudo nano /etc/systemd/timesyncd.conf

Comment out the line starting with FallbackNTP and replace default servers with:

NTP = 192.168.200.37

For example:

[Time]
 NTP=192.168.200.37
 FallbackNTP=0.us.pool.ntp.org 1.us.pool.ntp.org

Edit he configuration file for NTP: /etc/ntp.conf.

sudo nano /etc/ntp.conf

You can edit it to set a new server for time synchronization (lines beginning with "pool").

### 1.7.1 NTP commands

NTP has fewer commands than timedatectl as everything is in the configuration file.

But you can use this one to manage the ntp server daemon:

sudo service ntp status | start | stop | restart

### 1.7.2 NTPDate

NTPDate is an additional command we install and use to force time synchronization.

Check your current time delay compared to a server:

sudo ntpdate -q 0.us.pool.ntp.org

Fix the delay now:

The NTP daemon will fix the delay step by step.

But if you want to fix it now, you can use:

sudo service ntp stop

sudo ntpdate 0.us.pool.ntp.org

sudo service ntp start

You need to stop the NTP daemon before using ntpdate to free the port.

Determine NTP status with:

ntpq -p

or

ntpq -pn

## 1.8 Install Python Libs

install pip

_sudo apt-get install python3-pip_

_sudo pip install evdev_

later versions of Rapberry Pi OS 'externally manage' Pi libraries and require anything not upported by the OS to be installed in a virtual environment.  evdv *IS* supported so can be installed with APT:

_sudo apt install python3-evdev_

### 1.8.1 Configure Pi GPIO Support

In order to use the GPIO ports your user must be a member of the gpio group. 
The pi user is a member by default, other users need to be added manually. To add a user:

  sudo usermod -a -G gpio keymaster

### 1.8.2 Piface DigitalIO 2 Support

PiFace is _ **only** _ used on legacy systems as the PiFace DigitalIO 2 cards are no longer available.

_pip install pifacecommon_

_pip install pifacedigitalio_

## 1.9 Install UDEV Rules

Copy the latest version of the UDEV rules file to /etc/udev/rules.d directory. The file should be named "10-local.rules"

## 1.10 Keymaster Install

Copy Python code and subdirectories into /home/Keymaster

Add KeyMaster.ini file

## 1.11 Bash Script Edits

The ".bashrc" needs some additions

Copy line(s) from aux\_files\bashrc-addendum.bash into the end of .bashrc

Copy file aux\_files\KeyMaster\_Start.sh to \home\keymaster and make executable with

_sudo chmod +x KeyMaster\_Start.sh_

## 1.12 Network Configuration

Need to modify network manager settings, so edit:

_/etc/dhcpcd.conf_

And set the appropriate static address

Note: Using Network Manager leads to host of issues that make copying the card image a mess. Network Manager includes the ethernet interface mac address in a GUID used a number of places, not all of which appear to be documented.

### 1.12.1 WiFi Configuration

If the applicaiton uses WiFi use the command:

_sudo nmtui_

to configure WiFi and IP settings with the appropriate static address. Some versions of Raspberry Pi OS need options disabled to maintain a reliable connection. If you observe frequent data drops try:

Create a file called /etc/modprode.d/brcmfmac.conf

 with the following content:

_options brcmfmac roamoff=1 feature_disable=0x82000_

Reference: https://forums.raspberrypi.com/viewtopic.php?t=372254

### 1.12.2  SSHD Configuration

The OpenSSH daemon has added QoS metric requirements that the Pi Zero W Wifi can't meet. This results in dropped SSH/SFTP sessions, often before login completes. Edit 

_/etc/ssh/sshd_config_

and add the line

_IPQoS 0x00_

Refernce: https://raspberrypi.stackexchange.com/questions/143142/has-anyone-solved-raspberry-pi-zero-w-ssh-client-loop-send-disconnect-broken

## 1.13 Read Only Card Configuration

While we have examples of systems running cards with local logging to SD for years, Flash wearout is a well-documented issue. The best option seems to be to log to local RAMDISK and network logger.

Add instructions here.

[1](#sdfootnote1anc) While cards smaller than 8 GiB may fit an image, smaller cards don't really leave enough working room for flash wear leveling.
