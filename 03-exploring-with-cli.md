# Exploring the Network with the NSO CLI 
Now that NSO is installed, we have a running instance, and it's connected to our network, let's see what we can do with it.  

## Device Configuration 
These first exercises will explore device configuration through NSO.  We'll look at how NSO can tell you about the current configuration, as well as how you can update the configuration across your network with ease.  

### Reading Device Configuration 
When you did the initial `sync-from` the network NSO retrieved all the current configurations of the devices and stored them in the "CDB".  This is NSO's Configuration Database, and the heart of the "magic" that makes NSO so powerful.  With the CDB, you now have access to the entire network's configuration from a single point through the NSO CLI, web ui, or through the variety of APIs NSO makes available. 

Even more exciting, this configuration isn't just another text file backup, but rather a fully modeled configuration.  If that didn't **WOW** you, just wait!

1. Start out by accessing the `ncs_cli` 

    ```bash
    ncs_cli -C -u admin
    ```

1. Like many CLI's, the NSO CLI allows for TAB completion, so as you enter commands make copious use of the TAB key.  
1. We'll start out simple, just looking at the full running configuration for a single device `leaf01-1`. 

    ```text
    show running-config devices device leaf01-1 config
    ```

    <details><summary>Partial Output</summary>

    ```
    devices device leaf01-1
    config
    tailfned default-lacp-suspend-individual true
    version  9.2(2) Bios:version
    hostname leaf01-1
    feature hsrp
    feature ospf
    feature vpc
    feature interface-vlan
    feature lacp
    no feature ssh
    feature telnet
    cfs eth distribute
    username admin password 5 $1$KuOSBsvW$Cy0TSD..gEBGBPjzpDgf51 role network-admin
    username cisco password 5 $1$Nk7ZkwH0$fyiRmMMfIheqE3BqvcL0C1 role network-operator
    username cisco role network-admin
    no ip domain-lookup
    mac address-table notification mac-move
    rmon event 1 description FATAL(1) owner PMON@FATAL
    rmon event 2 description CRITICAL(2) owner PMON@CRITICAL
    rmon event 3 description ERROR(3) owner PMON@ERROR
    rmon event 4 description WARNING(4) owner PMON@WARNING
    rmon event 5 description INFORMATION(5) owner PMON@INFO
    no password strength-check
    vlan 1
    !
    vlan 10
    name trusted-transit
    !
    vlan 11
    name admin-systems
    !
    vlan 12
    name mgmt
    !
    vlan 100
    name dmz-transit
    !
    vlan 101
    name prod-web
    !
    vlan 102
    name prod-app
    !
    vlan 103
    name prod-data
    !
    vlan 201
    name dev-web
    !
    vlan 202
    name dev-app
    !
    vlan 203
    name dev-data
    !
    vlan 900
    name guest-transit
    !
    vlan 901
    name hotspot
    !
    vlan 3001
    name backdoor-temp
    !
    vrf context management
    ip route 0.0.0.0/0 172.16.30.254
    exit
    vrf context backdoor
    exit
    vrf context dmz
    ip route 0.0.0.0/0 10.225.250.4
    exit
    .
    .
    .
    ```
    </details>

1. This command should seem very familiar to any network engineer.  Let's look at a couple of key details about the command.  
    * `show` - you're looking to "see" some info about the system.  No surprise here. 
    * `running-config` - you're interested in the current "running configuration" from the CDB within NSO.  This includes details about devices, but also about NSO itself. 
    * `devices device leaf01-1` - we don't want the full running-configuration, but rather we are interested in a specific device.  Each command following `running-config` drills into NSO's configuration, scoping the returned results.  
        * `devices` - we are looking for details about devices. 
        * `device` - we are looking for details about a specific device. 
            * *Note: device `authgroups` also are under the `devices` keyword.*
        * `leaf01-1` is the device we want details on. 
    * `config` - this is likely the most confusing bit to think about.  This tells NSO that we want information about the devices configuration, but you might wonder what else there is.  The answer is the details about how NSO connects to and communicates with the device.  The `address`, `authgroup`, `ned-settings` all are part of NSO's details for a device. 

1. That last point deserves a moment to explore.  Run this command and see what else NSO has for a device. 

    ```
    show running-config devices device leaf01-1 | de-select config
    ```

    > Note: the `de-select` keyword is a great way to remove unneeded elements of returned data. 

    <details><summary>Output</summary>

    ```
    devices device leaf01-1
    address   172.16.30.103
    ssh host-key-verification none
    authgroup labadmin
    device-type cli ned-id cisco-nx-cli-5.13
    device-type cli protocol ssh
    state admin-state unlocked
    !
    ```
    </details>

1. What you should see is the same configuration that we used to add this device into the NSO inventory, pretty neat huh!  

1. Let's use the power of the modeled config a bit more with these next exercises.  
    * Retrieve the list of vlans configured on the device 

        ```
        show running-config devices device leaf01-1 config nx:vlan
        ```

        <details><summary>Output</summary>

        ```
        devices device leaf01-1
        config
        vlan 1
        !
        vlan 10
        name trusted-transit
        !
        vlan 11
        name admin-systems
        !
        vlan 12
        name mgmt
        !
        vlan 100
        name dmz-transit
        !
        vlan 101
        name prod-web
        !
        vlan 102
        name prod-app
        !
        vlan 103
        name prod-data
        !
        vlan 201
        name dev-web
        !
        vlan 202
        name dev-app
        !
        vlan 203
        name dev-data
        !
        vlan 900
        name guest-transit
        !
        vlan 901
        name hotspot
        !
        vlan 3001
        name backdoor-temp
        ```
        </details>    

        > The `nx:vlan` highlights the use of NEDs to interact with the devices.  Each type of device, and therefore NED, has it's own data model for configuration and operational data.  The `nx:` indicates the NED, or specifically the YANG model, that is used for the data.  

    * What if you wanted the data in a format better for programmability.. let's say JSON.  

        ```
        show running-config devices device leaf01-1 config nx:vlan | display json
        ```

        <details><summary>Output</summary>

        ```
        {
        "data": {
            "tailf-ncs:devices": {
            "device": [
                {
                "name": "leaf01-1",
                "config": {
                    "tailf-ned-cisco-nx:vlan": {
                    "vlan-list": [
                        {
                        "id": 1
                        },
                        {
                        "id": 10,
                        "name": "trusted-transit"
                        },
                        {
                        "id": 11,
                        "name": "admin-systems"
                        },
                        {
                        "id": 12,
                        "name": "mgmt"
                        },
                        {
                        "id": 100,
                        "name": "dmz-transit"
                        },
                        {
                        "id": 101,
                        "name": "prod-web"
                        },
                        {
                        "id": 102,
                        "name": "prod-app"
                        },
                        {
                        "id": 103,
                        "name": "prod-data"
                        },
                        {
                        "id": 201,
                        "name": "dev-web"
                        },
                        {
                        "id": 202,
                        "name": "dev-app"
                        },
                        {
                        "id": 203,
                        "name": "dev-data"
                        },
                        {
                        "id": 900,
                        "name": "guest-transit"
                        },
                        {
                        "id": 901,
                        "name": "hotspot"
                        },
                        {
                        "id": 3001,
                        "name": "backdoor-temp"
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

        > `display xml` also works! 
    
    * What about the VLAN interfaces? 

        ```
        show running-config devices device leaf01-1 config nx:interface Vlan
        ```

        <details><summary>Partial Output</summary>

        ```
        devices device leaf01-1
        config
        interface Vlan1
        no shutdown
        no ip redirects
        no ipv6 redirects
        exit
        interface Vlan10
        no shutdown
        vrf member trusted
        ip address 10.225.250.2/29
        ip ospf passive-interface false
        no ip redirects
        ip router ospf 1 area 0.0.0.0
        no ipv6 redirects
        hsrp version 2
        hsrp 1 ipv4
            ip 10.225.250.1
            preempt
            priority 110
        exit
        exit
        interface Vlan11
        no shutdown
        vrf member trusted
        ip address 10.225.1.2/24
        no ip redirects
        ip router ospf 1 area 0.0.0.0
        no ipv6 redirects
        hsrp version 2
        hsrp 1 ipv4
            ip 10.225.1.1
            preempt
            priority 110
        exit
        exit
        interface Vlan12
        no shutdown
        vrf member trusted
        ip address 10.225.2.2/24
        no ip redirects
        ip router ospf 1 area 0.0.0.0
        no ipv6 redirects
        hsrp version 2
        hsrp 1 ipv4
            ip 10.225.2.1
            preempt
            priority 110
        exit
        exit
        .
        .
        ```
        </details>        

    * This one should wow you... what about just the IP addresses for all SVIs.

        ```
        show running-config devices device leaf01-1 config nx:interface Vlan * ip address
        ```

        <details><summary>Output</summary>

        ```
        devices device leaf01-1
        config
        interface Vlan10
        ip address 10.225.250.2/29
        exit
        interface Vlan11
        ip address 10.225.1.2/24
        exit
        interface Vlan12
        ip address 10.225.2.2/24
        exit
        interface Vlan100
        ip address 10.225.250.34/29
        exit
        interface Vlan101
        ip address 10.225.9.2/24
        exit
        interface Vlan102
        ip address 10.225.10.2/24
        exit
        interface Vlan103
        ip address 10.225.11.2/24
        exit
        interface Vlan201
        ip address 10.225.17.2/24
        exit
        interface Vlan202
        ip address 10.225.18.2/24
        exit
        interface Vlan203
        ip address 10.225.19.2/24
        exit
        interface Vlan900
        ip address 10.225.250.50/29
        exit
        interface Vlan901
        ip address 10.225.49.2/24
        exit
        interface Vlan3001
        ip address 172.16.30.2/24
        exit
        ```
        </details>

        > NSO allows the use of wildcards in commands.  We'll use this more later. 

### Updating Device Configuration 
Now that you've seen how NSO can be used to read the current network configuration, let's see how we can update it.  

1. As always, start out at the `ncs_cli`. 

    ```
    ncs_cli -C -u admin
    ```

1. Since we'll be updating the configuration, we need to go into `config` mode. 

    ```
    config 
    ```

1. Notice that the prompt changes in an expected way to indicate our new mode. 

    ```
    Entering configuration mode terminal
    admin@ncs(config)#
    ```

1. From here we can change the configuration for anything in NSO, but we want to update a devices configuration so we'll need to move into that device.  

    ```
    devices device leaf01-1
    ```

1. Again, look at the prompt.  It will help you understand where you are in context.  

    ```
    admin@ncs(config-device-leaf01-1)# 
    ```

1. Remember from our prior exercise that the device configuration is under the `config` section in the device model.  So we'll enter another `config` to indicate we'll be configuring the device. 

    ```
    config
    ```

1. The prompt once again changes (this is the last time I'll mention it, but it's important to keep an eye on it for clues as to where you're current working.)

    ```
    admin@ncs(config-config)#
    ```

1. Rather than continuing to add additional keywords to the prompt, NSO shortens it here to just `config-config` to indicate a nested config mode.  This is to keep the prompt reasonable in size.  But if you ever want to know "where" you are, NSO takes inspiration from the "print working directory" `pwd` command from Linux.  Use it to see where you are.  

    ```
    pwd
    ```

    <details><summary>Output</summary>

    ```
    Current submode path:
    devices device leaf01-1 \ config
    ```
    </details>

    > This is a VERY handy way to assist with navigation. 

1. Okay, let's get to that configuration change.  And what's better for a configuration change in the network than adding a new VLAN and interface!  Enter these commands. 

    > Don't forget you can use TAB completion.  And notice the use of `nx:` in the configuration to indicate the NED or device type we are working with. 

    ```
    nx:vlan 42
    name TheAnswer
    exit
    nx:interface Vlan 42 
    description "Answer to the Ultimate Question of Life, the Universe, and Everything"
    ip address 10.42.42.42/24
    exit
    ```

1. What we've done is stage some new configuration within NSO, but it isn't actually "live" yet.  To make it live we need to `commit` it to the CDB, which will then push the updates out to the network.  Before we do that, let's checkout another very handy feature of NSO.

1. First jump to the `top` of `config` mode.  

    ```
    top
    ```

    > You could just type `exit` over and over again, but `top` is so much faster. 

1. Now let's look at the current configuration that is waiting to be committed. 

    ```
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    devices device leaf01-1
     config
      vlan 42
       name TheAnswer
      !
      interface Vlan42
       no shutdown
       description Answer to the Ultimate Question of Life, the Universe, and Everything
       ip address 10.42.42.42/24
      exit
    ```
    </details>

1. This is a very handy way to check your work before you commit it.  And suppose you were doing some work to prepare for a larger automation project where you wanted to make this change using the NETCONF or RESTCONF API for NSO... you can see what the XML version of this new configuration is using the `display xml` option and see the complete payload for an RPC `<edit>`

    ```
    show configuration | display xml
    ```

    <details><summary>Output</summary>

    ```xml
    <devices xmlns="http://tail-f.com/ns/ncs">
    <device>
        <name>leaf01-1</name>
        <config>
        <vlan xmlns="http://tail-f.com/ned/cisco-nx">
            <vlan-list>
            <id>42</id>
            <name>TheAnswer</name>
            </vlan-list>
        </vlan>
        <interface xmlns="http://tail-f.com/ned/cisco-nx">
            <Vlan>
            <name>42</name>
            <description>Answer to the Ultimate Question of Life, the Universe, and Everything</description>
            <ip>
                <address>
                <ipaddr>10.42.42.42/24</ipaddr>
                </address>
            </ip>
            </Vlan>
        </interface>
        </config>
    </device>
    </devices>
    ```
    </details>

1. One last question you might have before you're ready to `commit` this to the network, is what will NSO actually SEND to your device?  NSO lets you do a `dry-run` before committing, and it has a handy `native` format that shows you exactly what will be sent away.  

    ```
    commit dry-run outformat native
    ```

    <details><summary>Output</summary>

    ```
    native {
        device {
            name leaf01-1
            data vlan 42
                  name TheAnswer
                 !
                 interface Vlan42
                  no shutdown
                  description Answer to the Ultimate Question of Life, the Universe, and Everything
                  ip address 10.42.42.42/24
                exit
        }
    }
    ```
    </details>

1. And now we are ready to make the change.  

    ```
    commit
    ```

    <details><summary>Output</summary>

    ```
    Commit complete.
    ```
    </details>

1. You can see that all staged changes have been completed with another look. 

    ```
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    % No configuration changes found.
    ```
    </details>

#### Bonus: Rolling back configurations 
We've all been there... you make a network change and then immediately realize you made a mistake and need to undo it... but some changes are HUGE.  How do you roll it all back?  With NSO it's super easy.  

1. Let's see what it would take to rollback the latest change made.  

    ```
    show configuration rollback changes
    ```

    <details><summary>Output</summary>

    ```
    devices device leaf01-1
     config
      no vlan 42
      no interface Vlan42
    ```
    </details>

1. Now let's load up these changes.  Jump into config mode and setup the rollback.  

    ```
    rollback configuration
    ```

1. This "stages" those changes like any network change.  You can see it with `show configuration`. 

    <details><summary>Output</summary>

    ```
    devices device leaf01-1
     config
      no vlan 42
      no interface Vlan42
    ```
    </details>

1. And you can checkout what commands will be run on the device to accomplish this before committing. 

    ```
    commit dry-run outformat native 
    ```

    <details><summary>Output</summary>

    ```
    native {
        device {
            name leaf01-1
            data vlan 42
                no name
                !
                no vlan 42
                no interface Vlan42
        }
    }
    ```
    </details>

1. And complete the rollback

    ```
    commit
    ```

Pretty nice right!  A couple of key points about rollback. 

* NSO stores rollback files for every change, and you can load up a specific change by referencing the ID number of a change. 
* Each `commit` is a rollback.  So if a single commit updates 10 devices, and 30 configuration lines on each device, a rollback will undo all that configuration. 
* Depending on what has changed in the network since the commit you are trying to rollback, NSO may NOT be able to execute the rollback.  It will notify you of conflicts.  

### Device Templates


## Device Operational State

## NSO Live Status Device Interaction 