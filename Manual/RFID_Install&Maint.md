Revision 0v1 Dallas Makerspace Legacy RFID Lockout Installation and Maintenance

| Date | Revision | Description | Author |
| --- | --- | --- | --- |
| 15 JUL 23 | 0v1 | Initial Incomplete Draft | OZINDFW |
|
 |
 |
 |
 |
|
 |
 |
 |
 |

# Table of Contents

[1 Before You Begin 1](#_Toc146465683)

[1.1 Required Information 1](#_Toc146465684)

[1.2 Tools/Facilities 1](#_Toc146465685)

[2 Create SD card 1](#_Toc146465686)

[2.1 SD configuration 1](#_Toc146465687)

[2.1.1 Set hostname 1](#_Toc146465688)

[2.2 Password Setting 2](#_Toc146465689)

[2.3 Keymaster Configuration 2](#_Toc146465690)

[2.4 Network Configuration 2](#_Toc146465691)

[2.5 Labelling 2](#_Toc146465692)

[3 Installation 2](#_Toc146465693)

[3.1 Wiring 2](#_Toc146465694)

[3.1.1 Single Phase Application 2](#_Toc146465695)

[3.1.2 Three Phase Application 3](#_Toc146465696)

[4 Test and Calibration 3](#_Toc146465697)

[4.1 Current Sensor adjustment 3](#_Toc146465698)

[5 Maintenance 3](#_Toc146465699)

[Appendix A 4](#_Toc146465700)

[6 Configuration For Operation On 480 V (Untested) 4](#_Toc146465701)

| **DO NOT configure for operation at input voltage greater than 240V.** If 480V operation is required, see Appendix A; Configuration for operation on 480 V (Untested) |
| --- |

# 1Before You Begin

## 1.1Material Required

- Interlock hardware for configuration.
- Suitable micro SD card � minimum of 4 GiB, recommend 16 GiB or greater to provide lots of spare capacity for wear leveling.

## 1.2Required Information

Have the following information at hand:

- Hostname
- Fixed IP address for unit (ping hostname while at DMS to obtain.)
- The Active Directory security group that controls the target device (these are continuously added to and changing, but one example is "Automotive 102 (Lift Training)" for the Automotive Committee car lift.
- The password for the device. Generating the password with and storing it in the DMS Bitwarden IoT folder before using it would be a good idea.
- Required operating voltage
- Phase count (singe or three phase)
- Load current or horsepower rating

## 1.3Tools/Facilities

Have access to:

- SSH/SFTP terminal
- Multimeter
- Test supply of required voltage
- DMS Ethernet port on VLAN 400 (the IoT VLAN)

# 2Create and Configure SD card

Copy the latest image on to a micro SD or create one according to the instructions in the latest version of RFID\_LO\_SD\_IMAGE\_Setup

## 2.1SD configuration

You can do this from the autologin console on the Pi or by SSH to the default image DNS of Interlock-Test-1 with the password dmsDMS. If you use SSH you will need a network connection on VLAN 400.

### 2.1.1Resize Partition

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

#### 2.1.2.1Hostname Selection

Three elements separated by hyphens, all caps.

COMMITTEE-MACHINE-INDEX

**COMMITTEE** name - Wood, Machine, Metal, etc

**MACHINE** name � eg SAWSTOP for SawStop table saw, or COLCHESTER for Colchester lathe

**INDEX** � a sequential number starting with one and continuing with upper case letters if needed

So the woodshop's second SawStop would be WOOD-SAWSTOP-2

## 2.2Password Setting

Set new passwords for the Keymaster User account in accordance with DMS password selection standards and enter it in Bitwarden in the IoT folder for the hostname.

## 2.3Keymaster Configuration

Select AD name for authorization. Edit the [ADApiAuth] section of /home/KeyMaster/KeyMaster.ini

[ADApiAuth]

url = http://192.168.200.32:8080/api/v1/lookupByRfid

# Groups denied are checked first, then groups allowed are checked

groups\_denied =

groups\_allowed = Automotive 102 (Lift Training)

and replace the groups\_allowed entry with the AD for the system. groups\_denied should remain blank.

## 2.4Network Configuration

It is probably best to leave this to the very last as interlocks at DMS use fixed IPs on a dedicated VLAN subnet. The VLAN is 400 and is determined by switch port configuration. The subnet is the RFC1918 non-routable, private range 10.0.0.0

# 1
. The IoT IP range is not routable by the DMS user subnet.

Use the Network Manager command line interface to set the IP address.

Need to modify network manager settings, so run:

sudo nmcli con mod "Wired connection 1" ipv4.addresses "10.0.16.xx/21"

Where xx is the last octet of the IP address.

If you need to redo settings run:

sudo nmcli con mod "Wired connection 1" ipv4.addresses "10.0.16.10/21" ipv4.gateway "10.0.16.1" ipv4.dns "192.168.200.27" ipv4.method "manual"

Another option is to edit the settings file in /etc/NetworkManager/system-connections/ and manually change the IP address. While editing it would be a good idea to verify the other settings.

## 2.5Labelling

- Apply High Voltage Caution label if not present
- Apply hostname label to cabinet (best to use top of cabinet so label is visible with door open
- Apply hostname label to baseplate

# 3Installation

- Remove ground screw and four mounting nuts on baseplate (need pic) and remove the baseplate assembly.
- Select and remove cable entry and exit knockouts or cut holes for this purpose.
- Mount box/cabinet
- Route input and output cables
- Reinstall Baseplate
- Install (but do not tighten) ground screw.
- Place all corner nuts finger tight, then tighten them completely
- Tighten the ground screw

## 3.1Wiring

### 3.1.1Single Phase Application

In single phase applications the neutral should use the contactor middle terminals. The line (hot) should pass through the current sensor.

### 3.1.2Three Phase Application

# 4Test and Calibration

## 4.1Current Sensor adjustment

Turn the machine in it's unloaded running state (with motor on, but unloaded.) Adjust current switch so light just comes on. Turn motor off and verify sensor light turns off.

# 5Maintenance

Log checks

# Appendix A

# 6Configuration For Operation On 480 V (Untested)

**NOTE THAT THIS CONFIGURAITON IS UNTESTED**

The low voltage power supply input is rated for a maximum input operating voltage of 240 V. It should be possible to use the transformer input as an autoformer. Connect the power supply input between the common and 208V tap. Connect the 480 V input between the common and 480V tap.

[1](#sdfootnote1anc) 10.0.0.0 - 10.255.255.255 (10/8 prefix)

RackMultipart20231024-1-upk06t.docx Page 8 of 8[Publish Date]