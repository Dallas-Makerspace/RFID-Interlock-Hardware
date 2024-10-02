Dallas Makerspace Legacy RFID Lockout Installation and Maintenance
|

| **DO NOT configure for operation at input voltage greater than 240V.** If 480V operation is required, see Appendix A; Configuration for operation on 480 V (Untested) |
| --- |

# 1 Before You Begin

Note that the RFID Interlocks as currently built and tested have not been modified for power in/out ports, any indicators, or an Emergency Stop switch as these are installation dependent.  You will need to survey and plan the installation so that any required chassis modifications can be made prior to installation and commissioning. These instructions do not cover these modifications nor do they discuss mounting the interlock chassis/box.

## 1.1 Material Required

- Interlock hardware for configuration.
- Suitable micro SD card – minimum of 4 GiB, recommend 16 GiB or greater to provide lots of spare capacity for wear leveling.
- Box mounting material (wall anchors, etc.)
- Double sided mounting tape for reader installation.

## 1.2 Required Information

Have the following information at hand:

- Hostname
- Fixed IP address for unit (ping hostname while at DMS to obtain.)
- The Active Directory security group that controls the target device (these are continuously added to and changing, but one example is "Automotive 102 (Lift Training)" for the Automotive Committee car lift.
- The password for the device. Generating the password with and storing it in the DMS Bitwarden IoT folder before using it would be a good idea.
- Required operating voltage
- Phase count (single or three phase)
- Load current or horsepower rating

## 1.3 Tools/Facilities

Have access to:

- SSH/SFTP terminal
- Multimeter
- Test supply of required voltage
- DMS Ethernet port on VLAN 400 (the IoT VLAN)
- Very small screwdrivers required for current switch adjustment and DIN connection block screws
- 1/4" flat blade screwdriver and #1 Phillips screwdriver
- 3/8" socket or nutdriver for installing the baseplate retaining nuts.

# 2 Create and Configure SD card

Copy the latest image on to a micro SD or create one according to the instructions in the latest version of RFID\_LO\_SD\_IMAGE\_Setup

## 2.1 SD configuration

You can do this from the autologin console on the Pi or by SSH to the default image DNS of Interlock-Test-1 with the password dmsDMS. If you use SSH you will need a network connection on VLAN 400.

### 2.1.1 Resize Partition

Run the Raspberry Pi configuration utility and resize the partition to fill the card.

sudo raspi-config

select Advanced Options, and option A1 Expand Filesystem

### 2.1.2 Set hostname

Edit the /etc/hostname file:

sudo nano /etc/hostname

and enter the desired hostname. Edit the /etc/hosts

sudo nano /etc/hosts

file to change the localhost home alias entry (probably the last in the file) from:

127.0.0.1 Interlock-Test-1

To reflect the new hostname.

#### 2.1.2.1 Hostname Selection

Three elements separated by hyphens, all caps.

COMMITTEE-MACHINE-INDEX

**COMMITTEE** name - Wood, Machine, Metal, etc

**MACHINE** name – eg SAWSTOP for SawStop table saw, or COLCHESTER for Colchester lathe

**INDEX** – a sequential number starting with one and continuing with upper case letters if needed

So the woodshop's second SawStop would be WOOD-SAWSTOP-2

## 2.2 Password Setting

Set new passwords for the Keymaster User account in accordance with DMS password selection standards and enter it in Bitwarden in the IoT folder for the hostname.

## 2.3 Keymaster Configuration

Select AD name for authorization. 

Edit the [ADCommonAPIAuth] section of /home/KeyMaster/KeyMaster.ini

[ADCommonAPIAuth]

url = http://192.168.203.30:8083/badgeGroupMembership

group =

(where the authorization group follows the equal sign and is not in quotes)

~~Edit the [ADApiAuth] section of /home/KeyMaster/KeyMaster.ini~~

~~[ADApiAuth]~~

~~url = http://192.168.200.32:8080/api/v1/lookupByRfid~~

~~# Groups denied are checked first, then groups allowed are checked~~

~~groups\_denied =~~

~~groups\_allowed = Automotive 102 (Lift Training)~~

~~and replace the groups\_allowed entry with the AD for the system. groups\_denied should remain blank.~~

## 2.4 Network Configuration

It is probably best to leave this to the very last as interlocks at DMS use fixed IPs on a dedicated VLAN subnet. The VLAN is 400 and is determined by switch port configuration. The subnet is the RFC1918 non-routable, private range 10.0.0.0 0 - 10.255.255.255 (10/8 prefix)

The IoT IP range is not routable by the DMS user subnet.

Copy dhcpcd.conf from the aux-files/Network folder to /etc/dhcpcd.conf and edit the static IP address

## 2.5 Labelling

- Apply High Voltage Caution label if not present
- Apply hostname label to cabinet (best to use top of cabinet so label is visible with door open
- Apply hostname label to baseplate

# 3 Installation

- Remove ground screw and four mounting nuts on baseplate (need pic) and remove the baseplate assembly.
- Select and remove cable entry and exit knockouts or cut holes for this purpose.
- Mount box/cabinet
- Route input and output cables
- Reinstall Baseplate
- Install (but do not tighten) ground screw.
- Place all corner nuts finger tight, then tighten them completely
- Tighten the ground screw

## 3.1 Wiring

### 3.1.1 Single Phase Application

In single phase applications the neutral should use the contactor middle terminals. The line (hot) should pass through the current sensor.

### 3.1.2 Three Phase Application

# 4 Test and Calibration

## 4.1 Current Sensor adjustment

Turn the machine in it's unloaded running state (with motor on, but unloaded.) Adjust current switch so light just comes on. Turn motor off and verify sensor light turns off.

# 5 Maintenance

Log checks

# Appendix A

# 6 Configuration For Operation On 480 V (Untested)

**NOTE THAT THIS CONFIGURAITON IS UNTESTED**

The low voltage power supply input is rated for a maximum input operating voltage of 240 V. It should be possible to use the transformer input as an autoformer. Connect the power supply input between the common and 208V tap. Connect the 480 V input between the common and 480V tap.
