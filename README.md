# manageSwitch
Ugly bash script to do some basic operations on HP ProCurve 2848 switches. Might work on others aswell.


![Getting an interface status](https://www.ram.rwth-aachen.de/images/files/github_getState.png)

This script will manage our HP ProCurve 2484 switches.
You can
- change interface status (up, down)
- set a VLAN
- set the interface speed
- reset an intrusion flag.

Since v0.3 you can

- search all switches for a given mac address
- search all switches for a given ip address *

\* This requires setting ```SWC4K``` to your central router which has the ip->mac mapping in it's arp table, accessible via SNMP.

```Note: This release uses SNMPv3 to access the HP Switches, SNMPv2 for the corerouter (for arptable)```

```The SWC4k variable has to be there. It has to be the router with the arp table. If that is not properly available you will get errors like```

    No log handling enabled - using stderr logging
    getaddrinfo: .1.3.6.1.2.1.3.1.1.2.38.1 nodename nor servname provided, or not known
    snmpwalk: Unknown host (.1.3.6.1.2.1.3.1.1.2.38.1)

If you are not using a central router with SNMP available arptable set the ```SWC4K``` to something, e.g 192.168.1.1. You will then only get an error like:

    Timeout: No Response from 192.168.1.1

## Version
0.4

## Tech

manageSwitch uses a number of dependencies to work properly:

* snmpget
* snmpset
* snmpwalk

Make sure they are in the PATH variable or manually set in the script.

## Installation

```bash
$ git clone https://github.com/dwhoop55/manageSwitch.git
$ cd ./manageSwitch
$ chmod +x ./manageSwitch
```

Edit the script to match your switches' IPs, SNMP communities and binary paths.

Search for ```# GIT: EDIT HERE ``` and edit to make it work for your infrastructure.

## Options
*   -h       Show this message
*   -f       Force, do not ask for confirmation. MAKE SURE YOU KNOW WHAT YOU DO!
*   -s       The switch to manage:

        swSample1        | Sample switch 1
        swSample2        | Sample switch 2
*   -p       The port to manage:
        
        1-46 (since 47 and 48 are uplinks)
*   -a       The action on the port

        getState    |(get the current state of a port)
        up          |(set adminStatus to up (1))
        down        |(set adminStatus to down (2))
        unSetFlag   |(set intrusionFlag to unset (2), 1=set)
        # Unsupported by Switch # setFlag     |(set intrusionFlag to set (1), 2=unset)
        setVLAN     |(requires -n to be set!)
        setSpeed    |(requires -g to be set!)
        searchMAC   |(requires -m to be set!)
        searchIP    |(requires -i to be set!)
        ipToMAC     |(requires -i to be set!)
*   -n       The new VLAN for the port (define your own):

        1000         |(user vlan [Will be set if no -n argument])
        1200         |(admin vlan)
        1300         |(blocked vlan)
*   -g       The new interface speed (in Mbit/s):
        
        10          |(Mbit/s)
        100         |(Mbit/s [Will be set if no -g argument])
        1000        |(Mbit/s)
*   -m       The MAC address in the format:
        
        "AA:BB:Cc:DD:eE:DD"  or  "Aa-Bb-CC-DD-EE-DD"
*   -i       The IP address in the format:
        
        "192.168.0.123"
*   -b       Provide output in program friendly form

Now you can start using the script.

## Examples

### Let's get an interface status
```bash
$ ./manageSwitch -s swSample1 -a getState -p 48
```
    ramnet HP ProCurve 2484 switch manager
              version 0.4

    Switch: swSample1 (IP: 10.10.9.42)
    Port:   48
    Action: getState
    --------------------------------------
    | AdminStatus:   up (1)
    | OperStatus:    up (1)
    | IntrusionFlag: not set (2)
    | Oper Speed:    1 Gbit/s
    | Admin Speed:   Auto-neg.
    | VLAN:          1

### Using the -b flag (program friendly output)
```bash
$ ./manageSwitch -s swSample1 -a getState -p 48 -b
```
    AdminStatus up
    OperStatus down
    IntrusionFlag no
    Speed 100
    VLAN 1000

### Searching a MAC
```bash
$ ./manageSwitch -a searchMAC -m D0:AB:06:8D:BB:AA
```
    ramnet HP ProCurve 2484 switch manager
              version 0.4

    Switch: ? (IP: ?)
    Port:   ?
    Action: searchMAC D0:AB:06:8D:BB:AA
    --------------------------------------
    This may take a while. Searching...
    10.10.9.43 knows D0:AB:06:8D:BB:AA - looking at other switches..
    Success! Found MAC address (D0:AB:06:8D:BB:AA - IP:192.168.74.15) on Switch 10.10.9.43 on interface 27
    Interface details:
     | AdminStatus:   up (1)
     | OperStatus:    up (1)
     | IntrusionFlag: not set (2)
     | Oper Speed:    100 Mbit/s
     | Admin Speed:   Auto-neg.
     | VLAN:          1000


### Searching an IP
```bash
$ ./manageSwitch -a searchIP -i 192.168.32.11
```
    ramnet HP ProCurve 2484 switch manager
                  version 0.4

    Switch: ? (IP: ?)
    Port:   ?
    Action: searchIP 192.168.32.11
    --------------------------------------
    192.168.32.11 -> E4:82:C6:E4:19:C6
    This may take a while. Searching...
    10.10.9.43 knows E4:82:C6:E4:19:C6 - looking at other switches..
    Success! Found MAC address (E4:82:C6:E4:19:C6 - IP:192.168.32.11) on Switch 10.10.9.43 on interface 21
    Interface details:
     | AdminStatus:   up (1)
     | OperStatus:    up (1)
     | IntrusionFlag: not set (2)
     | Oper Speed:    100 Mbit/s
     | Admin Speed:   Auto-neg.
     | VLAN:          1200


### Disable an interface
```bash
$ ./manageSwitch -s swSample1 -p 10 -a down
```
    ramnet HP ProCurve 2484 switch manager
                  version 0.4

    Switch: swSample1 (IP: 10.10.9.42)
    Port:   10
    Action: down
    --------------------------------------
    This will set Port 10 to down. Are you sure? [y/n]y
     | AdminStatus:   up (1)
     | OperStatus:    up (1)
     | AdminStatus:   down (2)
     | OperStatus:    down (2)
