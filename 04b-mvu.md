# Minimum Viable Use Case: Gathering MAC Address Tables Across the Network
As part of troubleshooting and just general network operations, it's often valuable to have a view of the MAC address tables across the network.  If you've a large network with hundreds of devices, this can be very time consuming to gather manually.  Let's see how NSO can help. 

## Use Case Description 
Our goal is to retrieve the MAC address tables from our NX-OS and IOS switches. 

## Cisco NSO Features Used 
* Operational data available and modeled with NEDs
* Arbitrary command execution with `live-status exec`

## Building the Use Case 
Device NEDs provide access to some operational data along with the configuration data.  The operational data available for a device is dependent on the NED, and not all NEDs support the same details.  

But remember that even if a NED doesn't model out some operational data natively, you can still use NSO to gather operational details by sending `show` commands out to any connected device centrally and capturing the output. 

### Step 1: Nexus Devices 
The `cisco-nx` NED models out the MAC address table, let's check it out. 

1. Log into NSO with `ncs_cli -C -u admin`
1. Start with just one device to see how it works.  

    ```
    show devices device leaf01-1 live-status mac address-table
    ```

    <details><summary>Output</summary>

    ```
    VLAN  MAC ADDRESS     TYPE    STATIC  AGE  SECURE  NTFY  PORT              
    ---------------------------------------------------------------------------
    -     5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    10    0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    10    5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    10    5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    100   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    100   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    100   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    101   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    101   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    101   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    102   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    102   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    102   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    103   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    103   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    103   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    11    0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    11    5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    11    5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    12    0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    12    5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    12    5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    201   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    201   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    201   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    202   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    202   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    202   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    203   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    203   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    203   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    3001  0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    3001  5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    3001  5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    900   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    900   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    900   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    901   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
    901   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
    901   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    ```
    </details>

1. Maybe you want it in JSON format instead. 

    ```
    show devices device leaf01-1 live-status mac address-table | display json
    ```

    <details><summary>Output</summary>

    ```json
    {
    "data": {
        "tailf-ncs:devices": {
        "device": [
            {
            "name": "leaf01-1",
            "live-status": {
                "tailf-ned-cisco-nx-stats:mac": {
                "address-table": [
                    {
                    "vlan": "-",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "10",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "10",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "10",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "100",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "100",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "100",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "101",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "101",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "101",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "102",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "102",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "102",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "103",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "103",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "103",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "11",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "11",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "11",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "12",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "12",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "12",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "201",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "201",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "201",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "202",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "202",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "202",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "203",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "203",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "203",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "3001",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "3001",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "3001",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "900",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "900",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "900",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    },
                    {
                    "vlan": "901",
                    "mac-address": "0000.0c9f.f001",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "901",
                    "mac-address": "5e00.0009.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "sup-eth1(R)"
                    },
                    {
                    "vlan": "901",
                    "mac-address": "5e00.000a.0007",
                    "type": "static",
                    "age": "-",
                    "secure": "F",
                    "ntfy": "F",
                    "port": "vPC Peer-Link(R)"
                    }
                ]
                }
            }
            }
        ]
        }
    }
    }
    ```
    </details>

1. And let's say you want it from **ALL** devices... 

    ```
    show devices device * live-status mac address-table
    ```

    > Note: this command can take some time to run as NSO is connecting to every relevant device, capturing the data, and then modeling it. 

    <details><summary>Output</summary>

    ```
    NAME        VLAN  MAC ADDRESS     TYPE    STATIC  AGE  SECURE  NTFY  PORT              
    ---------------------------------------------------------------------------------------
    dmz-sw01-1  -     5e00.0004.0007  static  -       -    F       F     sup-eth1(R)       
    dmz-sw01-2  -     5e00.0005.0007  static  -       -    F       F     sup-eth1(R)       
    dmz-sw02-1  -     5e00.0006.0007  static  -       -    F       F     sup-eth1(R)       
    dmz-sw02-2  -     5e00.0007.0007  static  -       -    F       F     sup-eth1(R)       
    leaf01-1    -     5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                10    0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                10    5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                10    5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                100   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                100   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                100   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                101   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                101   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                101   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                102   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                102   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                102   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                103   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                103   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                103   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                11    0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                11    5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                11    5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                12    0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                12    5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                12    5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                201   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                201   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                201   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                202   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                202   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                202   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                203   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                203   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                203   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                3001  0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                3001  5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                3001  5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                900   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                900   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                900   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
                901   0000.0c9f.f001  static  -       -    F       F     sup-eth1(R)       
                901   5e00.0009.0007  static  -       -    F       F     sup-eth1(R)       
                901   5e00.000a.0007  static  -       -    F       F     vPC Peer-Link(R)  
    leaf01-2    -     5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                10    0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                10    5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                10    5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                100   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                100   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                100   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                101   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                101   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                101   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                102   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                102   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                102   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                103   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                103   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                103   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                11    0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                11    5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                11    5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                12    0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                12    5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                12    5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                201   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                201   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                201   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                202   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                202   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                202   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                203   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                203   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                203   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                3001  0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                3001  5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                3001  5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                900   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                900   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                900   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
                901   0000.0c9f.f001  static  -       -    F       F     vPC Peer-Link(R)  
                901   5e00.0009.0007  static  -       -    F       F     vPC Peer-Link(R)  
                901   5e00.000a.0007  static  -       -    F       F     sup-eth1(R)       
    leaf02-1    -     5e00.000b.0007  static  -       -    F       F     sup-eth1(R)       
    leaf02-2    -     5e00.000c.0007  static  -       -    F       F     sup-eth1(R)       
    spine01-1   -     5e00.0012.0007  static  -       -    F       F     sup-eth1(R)       
    spine01-2   -     5e00.0013.0007  static  -       -    F       F     sup-eth1(R)   
    ```
    </details>

1. Often times you may want to capture and save this data for another system or just to store it for later.  You can easily save the output to a file. 

    ```
    show devices device * live-status mac address-table | save nso-mac-address-table.txt
    ```

    > Note: this command can take some time to run as NSO is connecting to every relevant device, capturing the data, and then modeling it. 

### Step 2: Cisco IOS Devices 
Unfortunately the `cisco-ios` NED doesn't include `mac address-table` with the `live-status` options (though there are many other options).  So for these devices we'll use the `exec` feature to send the command `show mac address-table` to the devices. 

1. Log into NSO with `ncs_cli -C -u admin`
1. Run this command to send our show command to a single device. 

    ```
    devices device vm-sw01 live-status exec show mac address-table
    ```

    <details><summary>Output</summary>

    ```
    result 
            Mac Address Table
    -------------------------------------------

    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    1    0026.f004.0000    DYNAMIC     Po2
    1    5e00.000b.000c    DYNAMIC     Po2
    1    5e00.000c.000c    DYNAMIC     Po2
    10    0000.0c9f.f001    DYNAMIC     Po2
    10    0026.f004.0000    DYNAMIC     Po2
    10    5e00.0009.0007    DYNAMIC     Po2
    10    5e00.000a.0007    DYNAMIC     Po2
    10    fa16.3e2f.1826    DYNAMIC     Po2
    11    0000.0c9f.f001    DYNAMIC     Po2
    11    0026.f004.0000    DYNAMIC     Po2
    11    5e00.000a.0007    DYNAMIC     Po2
    12    0000.0c9f.f001    DYNAMIC     Po2
    12    0026.f004.0000    DYNAMIC     Po2
    12    5e00.000a.0007    DYNAMIC     Po2
    100    0000.0c9f.f001    DYNAMIC     Po2
    100    0026.f004.0000    DYNAMIC     Po2
    100    5e00.0009.0007    DYNAMIC     Po2
    100    5e00.000a.0007    DYNAMIC     Po2
    100    fa16.3e2f.1826    DYNAMIC     Po2
    101    0000.0c9f.f001    DYNAMIC     Po2
    101    0026.f004.0000    DYNAMIC     Po2
    101    5e00.000a.0007    DYNAMIC     Po2
    101    fa16.3e34.7de2    DYNAMIC     Gi1/0
    102    0000.0c9f.f001    DYNAMIC     Po2
    102    0026.f004.0000    DYNAMIC     Po2
    102    5e00.000a.0007    DYNAMIC     Po2
    102    fa16.3e20.7d11    DYNAMIC     Gi1/3
    103    0000.0c9f.f001    DYNAMIC     Po2
    103    0026.f004.0000    DYNAMIC     Po2
    103    5e00.000a.0007    DYNAMIC     Po2
    201    0000.0c9f.f001    DYNAMIC     Po2
    201    0026.f004.0000    DYNAMIC     Po2
    201    5e00.000a.0007    DYNAMIC     Po2
    202    0000.0c9f.f001    DYNAMIC     Po2
    202    0026.f004.0000    DYNAMIC     Po2
    202    5e00.000a.0007    DYNAMIC     Po2
    202    fa16.3e8c.4ba9    DYNAMIC     Gi1/2
    203    0000.0c9f.f001    DYNAMIC     Po2
    203    0026.f004.0000    DYNAMIC     Po2
    203    5e00.000a.0007    DYNAMIC     Po2
    203    fa16.3e03.c0fb    DYNAMIC     Gi1/1
    900    0000.0c9f.f001    DYNAMIC     Po2
    900    0026.f004.0000    DYNAMIC     Po2
    900    5e00.0009.0007    DYNAMIC     Po2
    900    5e00.000a.0007    DYNAMIC     Po2
    900    fa16.3e2f.1826    DYNAMIC     Po2
    901    0000.0c9f.f001    DYNAMIC     Po2
    901    0026.f004.0000    DYNAMIC     Po2
    901    5e00.000a.0007    DYNAMIC     Po2
    3001    0000.0c9f.f001    DYNAMIC     Po2
    3001    0026.f004.0000    DYNAMIC     Po2
    3001    5e00.000a.0007    DYNAMIC     Po2
    Total Mac Addresses for this criterion: 52
    vm-sw01#
    ```
    </details>

1. We can run this across multiple devices by using our wildcard option.  Our `cisco-ios` switches are named `vm-sw01` and `vm-sw02` giving us this option. 

    ```
    devices device vm-sw* live-status exec show mac address-table
    ```

    <details><summary>Output</summary>

    ```
    devices device vm-sw01 live-status exec show
        result 
            Mac Address Table
    -------------------------------------------

    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    1    0026.f004.0000    DYNAMIC     Po2
    1    5e00.000b.000c    DYNAMIC     Po2
    1    5e00.000c.000c    DYNAMIC     Po2
    10    0000.0c9f.f001    DYNAMIC     Po2
    10    0026.f004.0000    DYNAMIC     Po2
    10    5e00.0009.0007    DYNAMIC     Po2
    10    5e00.000a.0007    DYNAMIC     Po2
    10    fa16.3e2f.1826    DYNAMIC     Po2
    11    0000.0c9f.f001    DYNAMIC     Po2
    11    0026.f004.0000    DYNAMIC     Po2
    11    5e00.000a.0007    DYNAMIC     Po2
    12    0000.0c9f.f001    DYNAMIC     Po2
    12    0026.f004.0000    DYNAMIC     Po2
    12    5e00.000a.0007    DYNAMIC     Po2
    100    0000.0c9f.f001    DYNAMIC     Po2
    100    0026.f004.0000    DYNAMIC     Po2
    100    5e00.0009.0007    DYNAMIC     Po2
    100    5e00.000a.0007    DYNAMIC     Po2
    100    fa16.3e2f.1826    DYNAMIC     Po2
    101    0000.0c9f.f001    DYNAMIC     Po2
    101    0026.f004.0000    DYNAMIC     Po2
    101    5e00.0009.0007    DYNAMIC     Po2
    101    5e00.000a.0007    DYNAMIC     Po2
    101    fa16.3e34.7de2    DYNAMIC     Gi1/0
    102    0000.0c9f.f001    DYNAMIC     Po2
    102    0026.f004.0000    DYNAMIC     Po2
    102    5e00.0009.0007    DYNAMIC     Po2
    102    5e00.000a.0007    DYNAMIC     Po2
    102    fa16.3e20.7d11    DYNAMIC     Gi1/3
    103    0000.0c9f.f001    DYNAMIC     Po2
    103    0026.f004.0000    DYNAMIC     Po2
    103    5e00.0009.0007    DYNAMIC     Po2
    103    5e00.000a.0007    DYNAMIC     Po2
    201    0000.0c9f.f001    DYNAMIC     Po2
    201    0026.f004.0000    DYNAMIC     Po2
    201    5e00.000a.0007    DYNAMIC     Po2
    202    0000.0c9f.f001    DYNAMIC     Po2
    202    0026.f004.0000    DYNAMIC     Po2
    202    5e00.0009.0007    DYNAMIC     Po2
    202    5e00.000a.0007    DYNAMIC     Po2
    202    fa16.3e8c.4ba9    DYNAMIC     Gi1/2
    203    0000.0c9f.f001    DYNAMIC     Po2
    203    0026.f004.0000    DYNAMIC     Po2
    203    5e00.0009.0007    DYNAMIC     Po2
    203    5e00.000a.0007    DYNAMIC     Po2
    203    fa16.3e03.c0fb    DYNAMIC     Gi1/1
    900    0000.0c9f.f001    DYNAMIC     Po2
    900    0026.f004.0000    DYNAMIC     Po2
    900    5e00.0009.0007    DYNAMIC     Po2
    900    5e00.000a.0007    DYNAMIC     Po2
    900    fa16.3e2f.1826    DYNAMIC     Po2
    901    0000.0c9f.f001    DYNAMIC     Po2
    901    0026.f004.0000    DYNAMIC     Po2
    901    5e00.000a.0007    DYNAMIC     Po2
    3001    0000.0c9f.f001    DYNAMIC     Po2
    3001    0026.f004.0000    DYNAMIC     Po2
    3001    5e00.000a.0007    DYNAMIC     Po2
    Total Mac Addresses for this criterion: 57
    vm-sw01#
    devices device vm-sw02 live-status exec show
        result 
            Mac Address Table
    -------------------------------------------

    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    1    0026.f003.0000    DYNAMIC     Po2
    1    5e00.000b.000d    DYNAMIC     Po2
    1    5e00.000c.000d    DYNAMIC     Po2
    10    0000.0c9f.f001    DYNAMIC     Po2
    10    0026.f003.0000    DYNAMIC     Po2
    10    5e00.0009.0007    DYNAMIC     Po2
    10    5e00.000a.0007    DYNAMIC     Po2
    10    fa16.3e2f.1826    DYNAMIC     Po2
    11    0000.0c9f.f001    DYNAMIC     Po2
    11    0026.f003.0000    DYNAMIC     Po2
    11    5e00.000a.0007    DYNAMIC     Po2
    12    0000.0c9f.f001    DYNAMIC     Po2
    12    0026.f003.0000    DYNAMIC     Po2
    12    5e00.000a.0007    DYNAMIC     Po2
    100    0000.0c9f.f001    DYNAMIC     Po2
    100    0026.f003.0000    DYNAMIC     Po2
    100    5e00.0009.0007    DYNAMIC     Po2
    100    5e00.000a.0007    DYNAMIC     Po2
    100    fa16.3e2f.1826    DYNAMIC     Po2
    101    0000.0c9f.f001    DYNAMIC     Po2
    101    0026.f003.0000    DYNAMIC     Po2
    101    5e00.0009.0007    DYNAMIC     Po2
    101    5e00.000a.0007    DYNAMIC     Po2
    102    0000.0c9f.f001    DYNAMIC     Po2
    102    0026.f003.0000    DYNAMIC     Po2
    102    5e00.0009.0007    DYNAMIC     Po2
    102    5e00.000a.0007    DYNAMIC     Po2
    103    0000.0c9f.f001    DYNAMIC     Po2
    103    0026.f003.0000    DYNAMIC     Po2
    103    5e00.0009.0007    DYNAMIC     Po2
    103    5e00.000a.0007    DYNAMIC     Po2
    103    fa16.3e14.4bbf    DYNAMIC     Gi1/1
    201    0000.0c9f.f001    DYNAMIC     Po2
    201    0026.f003.0000    DYNAMIC     Po2
    201    5e00.000a.0007    DYNAMIC     Po2
    202    0000.0c9f.f001    DYNAMIC     Po2
    202    0026.f003.0000    DYNAMIC     Po2
    202    5e00.0009.0007    DYNAMIC     Po2
    202    5e00.000a.0007    DYNAMIC     Po2
    203    0000.0c9f.f001    DYNAMIC     Po2
    203    0026.f003.0000    DYNAMIC     Po2
    203    5e00.0009.0007    DYNAMIC     Po2
    203    5e00.000a.0007    DYNAMIC     Po2
    900    0000.0c9f.f001    DYNAMIC     Po2
    900    0026.f003.0000    DYNAMIC     Po2
    900    5e00.0009.0007    DYNAMIC     Po2
    900    5e00.000a.0007    DYNAMIC     Po2
    900    fa16.3e2f.1826    DYNAMIC     Po2
    901    0000.0c9f.f001    DYNAMIC     Po2
    901    0026.f003.0000    DYNAMIC     Po2
    901    5e00.000a.0007    DYNAMIC     Po2
    3001    0000.0c9f.f001    DYNAMIC     Po2
    3001    0026.f003.0000    DYNAMIC     Po2
    3001    5e00.000a.0007    DYNAMIC     Po2
    Total Mac Addresses for this criterion: 54
    vm-sw02#
    ```
    </details>

1. We can even just run the command against all devices... if a device doesn't support the command you'll get an error in the results.  

    > **Warning:** A bit of COA on this one... this option will send this command to every device.  Make sure you know what the devices in your network will do if given an unknown command before sending this away.  You've been warned!

    ```
    devices device * live-status exec show mac address-table
    ```

    <details><summary>Output</summary>

    ```
    devices device dmz-fw01-1 live-status exec show
        result 
                        ^
    ERROR: % Invalid input detected at '^' marker.
    dmz-fw01-1# 
    devices device dmz-rtr01-1 live-status exec show
        result 
    show mac address-table
            ^
    % Invalid input detected at '^' marker.

    dmz-rtr02-1#
    devices device dmz-rtr02-1 live-status exec show
        result 
    show mac address-table
            ^
    % Invalid input detected at '^' marker.

    dmz-rtr02-1#
    devices device dmz-rtr02-2 live-status exec show
        result 
    show mac address-table
            ^
    % Invalid input detected at '^' marker.

    dmz-rtr02-2#
    devices device dmz-sw01-1 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G    -     5e00.0004.0007   static   -         F      F    sup-eth1(R)

    devices device dmz-sw01-2 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G    -     5e00.0005.0007   static   -         F      F    sup-eth1(R)

    devices device dmz-sw02-1 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G    -     5e00.0006.0007   static   -         F      F    sup-eth1(R)

    devices device dmz-sw02-2 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G    -     5e00.0007.0007   static   -         F      F    sup-eth1(R)

    devices device fw-inside-sw01 live-status exec show
        result 
            Mac Address Table
    -------------------------------------------

    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    1    0026.f003.0000    DYNAMIC     Po2
    1    5e00.0009.0012    DYNAMIC     Po2
    1    5e00.000a.0012    DYNAMIC     Po2
    10    0000.0c9f.f001    DYNAMIC     Po2
    10    0026.f003.0000    DYNAMIC     Po2
    10    5e00.0009.0007    DYNAMIC     Po2
    10    5e00.000a.0007    DYNAMIC     Po2
    10    fa16.3e2f.1826    DYNAMIC     Gi0/3
    11    0000.0c9f.f001    DYNAMIC     Po2
    11    0026.f003.0000    DYNAMIC     Po2
    11    5e00.000a.0007    DYNAMIC     Po2
    12    0000.0c9f.f001    DYNAMIC     Po2
    12    0026.f003.0000    DYNAMIC     Po2
    12    5e00.000a.0007    DYNAMIC     Po2
    100    0000.0c9f.f001    DYNAMIC     Po2
    100    0026.f003.0000    DYNAMIC     Po2
    100    5e00.0009.0007    DYNAMIC     Po2
    100    5e00.000a.0007    DYNAMIC     Po2
    100    fa16.3e2f.1826    DYNAMIC     Gi0/3
    101    0000.0c9f.f001    DYNAMIC     Po2
    101    0026.f003.0000    DYNAMIC     Po2
    101    5e00.000a.0007    DYNAMIC     Po2
    102    0000.0c9f.f001    DYNAMIC     Po2
    102    0026.f003.0000    DYNAMIC     Po2
    102    5e00.000a.0007    DYNAMIC     Po2
    103    0000.0c9f.f001    DYNAMIC     Po2
    103    0026.f003.0000    DYNAMIC     Po2
    103    5e00.0009.0007    DYNAMIC     Po2
    103    5e00.000a.0007    DYNAMIC     Po2
    201    0000.0c9f.f001    DYNAMIC     Po2
    201    0026.f003.0000    DYNAMIC     Po2
    201    5e00.000a.0007    DYNAMIC     Po2
    202    0000.0c9f.f001    DYNAMIC     Po2
    202    0026.f003.0000    DYNAMIC     Po2
    202    5e00.0009.0007    DYNAMIC     Po2
    202    5e00.000a.0007    DYNAMIC     Po2
    203    0000.0c9f.f001    DYNAMIC     Po2
    203    0026.f003.0000    DYNAMIC     Po2
    203    5e00.0009.0007    DYNAMIC     Po2
    203    5e00.000a.0007    DYNAMIC     Po2
    900    0000.0c9f.f001    DYNAMIC     Po2
    900    0026.f003.0000    DYNAMIC     Po2
    900    5e00.0009.0007    DYNAMIC     Po2
    900    5e00.000a.0007    DYNAMIC     Po2
    900    fa16.3e2f.1826    DYNAMIC     Gi0/3
    901    0000.0c9f.f001    DYNAMIC     Po2
    901    0026.f003.0000    DYNAMIC     Po2
    901    5e00.000a.0007    DYNAMIC     Po2
    3001    0000.0c9f.f001    DYNAMIC     Po2
    3001    0026.f003.0000    DYNAMIC     Po2
    3001    5e00.000a.0007    DYNAMIC     Po2
    Total Mac Addresses for this criterion: 51
    fw-inside-sw01#
    devices device fw-outside-sw01 live-status exec show
        result 
            Mac Address Table
    -------------------------------------------

    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    1    0026.f002.0000    DYNAMIC     Po2
    1    5e00.0006.0012    DYNAMIC     Po2
    1    5e00.0007.0012    DYNAMIC     Po2
    11    0000.0c9f.f001    DYNAMIC     Po2
    11    001e.bd79.d8c0    DYNAMIC     Po2
    11    001e.bdf2.aac0    DYNAMIC     Po2
    11    0026.f002.0000    DYNAMIC     Po2
    11    fa16.3e48.13c3    DYNAMIC     Gi0/3
    Total Mac Addresses for this criterion: 8
    fw-outside-sw01#
    devices device fw01 live-status exec show
        result 
                ^
    ERROR: % Invalid input detected at '^' marker.
    fw01# 
    devices device fw02 live-status exec show
        result 
                ^
    ERROR: % Invalid input detected at '^' marker.
    fw02# 
    devices device internet-rtr live-status exec show
        result 
    show mac address-table
            ^
    % Invalid input detected at '^' marker.

    internet-rtr#
    devices device leaf01-1 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G   10     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G   11     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G   12     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  100     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  101     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  102     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  103     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  900     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  901     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G 3001     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  201     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  202     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G  203     0000.0c9f.f001   static   -         F      F    sup-eth1(R)
    G    -     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G   10     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G   11     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G   12     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  100     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  101     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  102     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  103     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  900     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  901     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G 3001     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  201     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  202     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G  203     5e00.0009.0007   static   -         F      F    sup-eth1(R)
    G   10     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G   11     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G   12     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  100     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  101     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  102     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  103     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  900     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  901     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G 3001     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  201     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  202     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)
    G  203     5e00.000a.0007   static   -         F      F    vPC Peer-Link(R)

    devices device leaf01-2 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G   10     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G   11     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G   12     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  100     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  101     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  102     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  103     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  900     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  901     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G 3001     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  201     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  202     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G  203     0000.0c9f.f001   static   -         F      F    vPC Peer-Link(R)
    G   10     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G   11     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G   12     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  100     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  101     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  102     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  103     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  900     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  901     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G 3001     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  201     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  202     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G  203     5e00.0009.0007   static   -         F      F    vPC Peer-Link(R)
    G    -     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G   10     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G   11     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G   12     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  100     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  101     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  102     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  103     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  900     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  901     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G 3001     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  201     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  202     5e00.000a.0007   static   -         F      F    sup-eth1(R)
    G  203     5e00.000a.0007   static   -         F      F    sup-eth1(R)

    devices device leaf02-1 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G    -     5e00.000b.0007   static   -         F      F    sup-eth1(R)

    devices device leaf02-2 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G    -     5e00.000c.0007   static   -         F      F    sup-eth1(R)

    devices device spine01-1 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G    -     5e00.0012.0007   static   -         F      F    sup-eth1(R)

    devices device spine01-2 live-status exec show
        result 
    Legend: 
            * - primary entry, G - Gateway MAC, (R) - Routed MAC, O - Overlay MAC
            age - seconds since last seen,+ - primary entry using vPC Peer-Link,
            (T) - True, (F) - False, C - ControlPlane MAC, ~ - vsan
    VLAN     MAC Address      Type      age     Secure NTFY Ports
    ---------+-----------------+--------+---------+------+----+------------------
    G    -     5e00.0013.0007   static   -         F      F    sup-eth1(R)

    devices device vm-sw01 live-status exec show
        result 
            Mac Address Table
    -------------------------------------------

    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    1    0026.f004.0000    DYNAMIC     Po2
    1    5e00.000b.000c    DYNAMIC     Po2
    1    5e00.000c.000c    DYNAMIC     Po2
    10    0000.0c9f.f001    DYNAMIC     Po2
    10    0026.f004.0000    DYNAMIC     Po2
    10    5e00.0009.0007    DYNAMIC     Po2
    10    5e00.000a.0007    DYNAMIC     Po2
    10    fa16.3e2f.1826    DYNAMIC     Po2
    11    0000.0c9f.f001    DYNAMIC     Po2
    11    0026.f004.0000    DYNAMIC     Po2
    11    5e00.000a.0007    DYNAMIC     Po2
    12    0000.0c9f.f001    DYNAMIC     Po2
    12    0026.f004.0000    DYNAMIC     Po2
    12    5e00.000a.0007    DYNAMIC     Po2
    100    0000.0c9f.f001    DYNAMIC     Po2
    100    0026.f004.0000    DYNAMIC     Po2
    100    5e00.0009.0007    DYNAMIC     Po2
    100    5e00.000a.0007    DYNAMIC     Po2
    100    fa16.3e2f.1826    DYNAMIC     Po2
    101    0000.0c9f.f001    DYNAMIC     Po2
    101    0026.f004.0000    DYNAMIC     Po2
    101    5e00.0009.0007    DYNAMIC     Po2
    101    5e00.000a.0007    DYNAMIC     Po2
    101    fa16.3e34.7de2    DYNAMIC     Gi1/0
    102    0000.0c9f.f001    DYNAMIC     Po2
    102    0026.f004.0000    DYNAMIC     Po2
    102    5e00.000a.0007    DYNAMIC     Po2
    103    0000.0c9f.f001    DYNAMIC     Po2
    103    0026.f004.0000    DYNAMIC     Po2
    103    5e00.0009.0007    DYNAMIC     Po2
    103    5e00.000a.0007    DYNAMIC     Po2
    201    0000.0c9f.f001    DYNAMIC     Po2
    201    0026.f004.0000    DYNAMIC     Po2
    201    5e00.000a.0007    DYNAMIC     Po2
    202    0000.0c9f.f001    DYNAMIC     Po2
    202    0026.f004.0000    DYNAMIC     Po2
    202    5e00.0009.0007    DYNAMIC     Po2
    202    5e00.000a.0007    DYNAMIC     Po2
    202    fa16.3e8c.4ba9    DYNAMIC     Gi1/2
    203    0000.0c9f.f001    DYNAMIC     Po2
    203    0026.f004.0000    DYNAMIC     Po2
    203    5e00.0009.0007    DYNAMIC     Po2
    203    5e00.000a.0007    DYNAMIC     Po2
    203    fa16.3e03.c0fb    DYNAMIC     Gi1/1
    900    0000.0c9f.f001    DYNAMIC     Po2
    900    0026.f004.0000    DYNAMIC     Po2
    900    5e00.0009.0007    DYNAMIC     Po2
    900    5e00.000a.0007    DYNAMIC     Po2
    900    fa16.3e2f.1826    DYNAMIC     Po2
    901    0000.0c9f.f001    DYNAMIC     Po2
    901    0026.f004.0000    DYNAMIC     Po2
    901    5e00.000a.0007    DYNAMIC     Po2
    3001    0000.0c9f.f001    DYNAMIC     Po2
    3001    0026.f004.0000    DYNAMIC     Po2
    3001    5e00.000a.0007    DYNAMIC     Po2
    Total Mac Addresses for this criterion: 55
    vm-sw01#
    devices device vm-sw02 live-status exec show
        result 
            Mac Address Table
    -------------------------------------------

    Vlan    Mac Address       Type        Ports
    ----    -----------       --------    -----
    1    0026.f003.0000    DYNAMIC     Po2
    1    5e00.000b.000d    DYNAMIC     Po2
    1    5e00.000c.000d    DYNAMIC     Po2
    10    0000.0c9f.f001    DYNAMIC     Po2
    10    0026.f003.0000    DYNAMIC     Po2
    10    5e00.0009.0007    DYNAMIC     Po2
    10    5e00.000a.0007    DYNAMIC     Po2
    10    fa16.3e2f.1826    DYNAMIC     Po2
    11    0000.0c9f.f001    DYNAMIC     Po2
    11    0026.f003.0000    DYNAMIC     Po2
    11    5e00.000a.0007    DYNAMIC     Po2
    12    0000.0c9f.f001    DYNAMIC     Po2
    12    0026.f003.0000    DYNAMIC     Po2
    12    5e00.000a.0007    DYNAMIC     Po2
    100    0000.0c9f.f001    DYNAMIC     Po2
    100    0026.f003.0000    DYNAMIC     Po2
    100    5e00.0009.0007    DYNAMIC     Po2
    100    5e00.000a.0007    DYNAMIC     Po2
    100    fa16.3e2f.1826    DYNAMIC     Po2
    101    0000.0c9f.f001    DYNAMIC     Po2
    101    0026.f003.0000    DYNAMIC     Po2
    101    5e00.0009.0007    DYNAMIC     Po2
    101    5e00.000a.0007    DYNAMIC     Po2
    102    0000.0c9f.f001    DYNAMIC     Po2
    102    0026.f003.0000    DYNAMIC     Po2
    102    5e00.000a.0007    DYNAMIC     Po2
    103    0000.0c9f.f001    DYNAMIC     Po2
    103    0026.f003.0000    DYNAMIC     Po2
    103    5e00.0009.0007    DYNAMIC     Po2
    103    5e00.000a.0007    DYNAMIC     Po2
    103    fa16.3e14.4bbf    DYNAMIC     Gi1/1
    201    0000.0c9f.f001    DYNAMIC     Po2
    201    0026.f003.0000    DYNAMIC     Po2
    201    5e00.000a.0007    DYNAMIC     Po2
    202    0000.0c9f.f001    DYNAMIC     Po2
    202    0026.f003.0000    DYNAMIC     Po2
    202    5e00.0009.0007    DYNAMIC     Po2
    202    5e00.000a.0007    DYNAMIC     Po2
    203    0000.0c9f.f001    DYNAMIC     Po2
    203    0026.f003.0000    DYNAMIC     Po2
    203    5e00.0009.0007    DYNAMIC     Po2
    203    5e00.000a.0007    DYNAMIC     Po2
    900    0000.0c9f.f001    DYNAMIC     Po2
    900    0026.f003.0000    DYNAMIC     Po2
    900    5e00.0009.0007    DYNAMIC     Po2
    900    5e00.000a.0007    DYNAMIC     Po2
    900    fa16.3e2f.1826    DYNAMIC     Po2
    901    0000.0c9f.f001    DYNAMIC     Po2
    901    0026.f003.0000    DYNAMIC     Po2
    901    5e00.000a.0007    DYNAMIC     Po2
    3001    0000.0c9f.f001    DYNAMIC     Po2
    3001    0026.f003.0000    DYNAMIC     Po2
    3001    5e00.000a.0007    DYNAMIC     Po2
    Total Mac Addresses for this criterion: 53
    vm-sw02#
    ```

    </details>

1. And you can save the output to a file like this. 

    ```
    devices device vm-sw* live-status exec show mac address-table | save nso-show-mac-command-output.txt
    ```

### Bonus Step: Gathering MAC Address Tables with a REST API 
Anything you can do through the CLI with NSO, you can do through the APIs (REST and NETCONF). Here's how you can gather this information using the RESTCONF API for NSO. 

```bash
curl -X GET \
  -u admin:admin \
  -H 'Accept: application/yang-data+json' \
  http://localhost:8080/restconf/data/tailf-ncs:devices/device=leaf01-1/live-status/tailf-ned-cisco-nx-stats:mac/address-table 
```

<details><summary>Output</summary>

```json
{
  "tailf-ned-cisco-nx-stats:address-table": [
    {
      "vlan": "-",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "10",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "10",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "10",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "100",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "100",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "100",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "101",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "101",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "101",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "102",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "102",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "102",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "103",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "103",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "103",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "11",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "11",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "11",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "12",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "12",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "12",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "201",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "201",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "201",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "202",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "202",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "202",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "203",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "203",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "203",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "3001",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "3001",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "3001",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "900",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "900",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "900",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    },
    {
      "vlan": "901",
      "mac-address": "0000.0c9f.f001",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "901",
      "mac-address": "5e00.0009.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "sup-eth1(R)"
    },
    {
      "vlan": "901",
      "mac-address": "5e00.000a.0007",
      "type": "static",
      "age": "-",
      "secure": "F",
      "ntfy": "F",
      "port": "vPC Peer-Link(R)"
    }
  ]
}
```
</details>

The RESTCONF API for NSO follows the RFC 8040 standard for building URLs from YANG models.  So in the above URL you'll see the full namespace references for the data models in use. These are: 

* `tailf-ncs:devices` - The top level container for all devices in NSO. 
* `tailf-ned-cisco-nx-stats:mac` - Here you see that the `mac` container is from the `cisco-nx-stats` YANG model. 
