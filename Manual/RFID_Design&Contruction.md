Dallas Makerspace RFID Lockout Design and Construction

# 1 Design

## 1.1 Avoided custom board design.

This is a no-production-volume design that will built in small quantites over years. Custom board designs were avoided because:

- They will become obsolete and unsupportable rapidly if extreme care is not taken in component selection, and likely even if that care is taken.
- The size and cost advantage is not great in the small number of large machine applications we have.

## 1.2 Readily Available Commodity Parts

The industrial parts used in this design and more expensive that their board level counterparts, but have longer market lives and available from several sources.

# 2 Fabrication and Construction

## 2.1 Waterjet Cutting Connector Hole

The inside of the box needs to protected from the jet. Even though the jet is scattered, it still contains enough energy to damage the internal powder coat. The rear and sidewall opposite the hinge need to be masked or protected in some way.

We've not tested an approach, but duct tape or an acrylic shield to protect the interior finish may work.

# 3 Versions

At this time there are _roughly_ three versions of Interlock. These are (informally) known as Legacy A, Legacy B, and DIN rail based designs.

The Legacy designs use an Eaton contactor with exposed electrical connections and have a protected high voltage compartment with a plastic cover that protects users from any voltage over 24 VAC.

Legacy A versions have the 5 VDC power supply mounted under the input side of the contactor. Only two seem to exist, one on the Shark Lathe in the Machine Shop and one on the Auto Lift. There are rumors of others.

Four Legacy B versions have been built and will likely end up on the Machine Shop milling machines, the Clausing lathe, and the Kalamazoo cold saw. The main apparent difference of this version is that the 5VDC power supply is mounted on the HV compartment wall.

The schematics of the two versions are similar, but not identical. The two legacy A variants are different as well. The Auto Lift unit has an RGB indicator mounted on it's top.

The DIN rail versions are based on a DIN rail mounted on 9.75" square plate.

# 4 Issues

## 4.1 Ethernet connection

Some Raspberry Pi Ethernet interfaces will not connect to some switches. This happens infrequently and can be resolved by connecting to a different brand or model switch. The cause of this issue has not been researched.

# 5 RFID Reader Identification

There are challenges identifying the RFID reader as the identification reported can vary a lot. Most readers do not seem to include a unique serial number. My hope was that future versions could use more than one reader to control multiple clients, unfortunately connecting multiple identical readers to a Raspberry PI only shows one reader. More study is needed. 

One solution was hoped to be to control reader identification by physical port. This is practical utilizing UDEV rules, but depends on underlying hardware. Raspberry Pi hardware versions may be determined by reading /proc/cpuinfo | Revision

[https://ozzmaker.com/check-raspberry-software-hardware-version-command-line/](https://ozzmaker.com/check-raspberry-software-hardware-version-command-line/)

**Table 1 Raspberry Pi Revision Codes**

| **Model and Pi Revision** | **256MB** | **Hardware Revision Code**** from cpuinfo** |
| --- | --- | --- |
| Model B Revision 1.0 | 256MB | 0002 |
| Model B Revision 1.0 + ECN0001 (no fuses, D14 removed) | 256MB | 0003 |
| Model B Revision 2.0
 Mounting holes | 256MB | 0004
 0005
 0006 |
| Model A
 Mounting holes | 256MB | 0007
 0008
 0009 |
| Model B Revision 2.0
 Mounting holes | 512MB | 000d
 000e
 000f |
| Model B+ | 512MB | 0010 |
| Compute Module | 512MB | 0011 |
| Model A+ | 256MB | 0012 |
| Pi 2 Model B | 1GB | a01041 (Sony, UK)
 a21041 (Embest, China) |
| PiZero | 512MB | 900092(no camera connector)
 900093(camera connector) |
| Pi 3 Model B Rev 1.2 | 1GB | a02082 (Sony, UK)
 a22082 (Embest, China) |
| Pi 3 Model B Plus Rev 1.3 | 1GB | a020d3 |
| PiZero W | 512MB | 9000c1 |
| Raspberry Pi 4 Model B Rev 1.1 | 4GB | c03111 |
|
| |
|
| |
| Data from several online sources and direct collection on devices in hand. |

We will need to add a udev rule that assigns reader(s) to port(s) and does so coarse verification of the connected device in an attempt to insure it's a reader.

Port identification and specification

[https://unix.stackexchange.com/questions/66901/how-to-bind-usb-device-under-a-static-name](https://unix.stackexchange.com/questions/66901/how-to-bind-usb-device-under-a-static-name)

udevadm info --attribute-walk /dev/ttyACM0

you get a screed of output but the salient bits are

KERNEL="video0", SUBSYSTEM="video4linux" and KERNELS="1:1.2.4:1.0".

# 6 RFID Reader I/O Assignment

We need to reliably locate the USB RFID reader. The commodity USB RFID readers are mostly based on the Sycreader chipset by RFID Technology Co., Ltd. The problem is that most are not serialized, and offer many different ID strings.

We also need to differentiate between readers in multi-contactor cases. The main advantage of multi-contactor installations is space savings as the cost savings are small.

Facing the the board edge, Pi 2 and 3 the Ethernet connection is on the left and the USB connections are on the right. The Pi 4 is a mirror image with USB ports in the center column.

![Shape1](RackMultipart20231024-1-vs8l2p_html_6112e96bd5550487.gif) ![Shape2](RackMultipart20231024-1-vs8l2p_html_39995c4b3256d1ef.gif) ![Shape3](RackMultipart20231024-1-vs8l2p_html_cfd2d0b7a700b3fc.gif)

RFID reader connections are outlined in red. Pi 4 USB 3.0 connections are shaded in blue.

To retain the top USB slots for other purposes, the bottom USB slots are used on Pi 2 and 3 devices.

On the Pi 4 one column of USB connections is version 2.0, and the other is version 3.0. Facing the Pi 4 board edge, the ethernet connector is on the right, the USB 2.0 connections are on the left (outlined in red) and the USB 3.0 connections (highlighted in blue) are in the middle. On a Pi 4 we will use the USB 2.0 connections for the reader to preserve the USB 3.0 connections for other use.

Regardless, we want to assign card readers device names RFIDx where x is 0-9

We will accomplish this with a udev rules file.

The file must be in the /etc/udev/rules.d directory and should be named "10-local.rules" ![](RackMultipart20231024-1-vs8l2p_html_39994beed71d8d95.gif)

The rules file is maintained on the DMS KeyMaster GitHub.

## 6.1 [How to bind USB device under a static name?](https://unix.stackexchange.com/questions/66901/how-to-bind-usb-device-under-a-static-name)

https://unix.stackexchange.com/questions/66901/how-to-bind-usb-device-under-a-static-name

### 6.1.1 Writing udev rules

[http://www.reactivated.net/writing\_udev\_rules.html](http://www.reactivated.net/writing_udev_rules.html)

udevadm info --attribute-walk /dev/ttyAMA0

finding card reader mounts

ls /dev/input/by-path

# 7 Auto restart scripts

A bug in early code versions caused reads of unauthorized tags to crash the software. An expedient work around was used in the form of a bash script that checked for the presence of the process every second. If the process was not active, it would restart KeyMaster. While this is no longer necessary, it's a useful tool in the event of emergent problems like power blips. The original script did not do useful logging, all that happened was that the restart of keymaster generated a private local log. We are in the process of converting all logging to syslog.

Bash scripts can use syslog with the 'logger' command

logger _Some message to write_

There are several options available, including:

-i Log the process ID in each line

-f Log the contents of a specified file

-n Write to the specified remote syslog server

-p Specify a priority

-t Tag the line with a specified tag

See man 1 logger for more information on the tool.

# 8 Alternate AD lookups

Joshua Freedman wrote a new/much simpler AD lookup

https://discord.com/channels/862552330322182165/862555939159539732/1128763838230175744

adm\_d4rkfly3r@prlxjump02:~$ curl -X GET \

-H "Content-type: application/json" \

-H "Accept: application/json" \

-d '{"username":"d4rkfly3r","group":"3D Printer Basics"}' \

"192.168.203.30:8083/usernameGroupMembership"

{"inGroup":true,"activeMember":true}

Or

curl -X GET \

-H "Content-type: application/json" \

-H "Accept: application/json" \

-d '{"badge":"XXXXXXXXXX","group":"Machine Shop Shark Lathe"}' \

"192.168.203.30:8083/badgeGroupMembership"

| **DO NOT copy and paste text directly from this document.**** Word inserts non-printing characters that are not reliably stripped.** |
| --- |

How to use with keyfob #/AUTH GROUP Looks pretty straightforward:

[https://github.com/Dallas-Makerspace/CommonApi/tree/master](https://github.com/Dallas-Makerspace/CommonApi/tree/master)

**Badge in group check**

GET /badgeGroupMembership

Content-Type: application/json

{

"badge": "",

"group": "",

"breakCache": "false"

}

breakCache is optional, and likely should always be left the default false. It is only being left in for debugging purposes. Which will return 200 OK or 417 Expectation failed (when an invalid ldap precondition fails). A 200 OK Body will have a result like below:

# 9 Dust Collector Blast Gate Controls

The blast gates look pretty simple overall, the basically turn into a contact closure. This is true either literally, as in the case of the iVac consumer units, or as a switched AC voltage in the case of more industrial units.

RackMultipart20231024-1-vs8l2p.docx Page 8 of 8[Publish Date]
