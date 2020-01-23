# Exploring the Network with the NSO CLI 
Now that NSO is installed, we have a running instance, and it's connected to our network, let's see what we can do with it.  

## Device Configuration 
These first exercises will explore device configuration through NSO.  We'll look at how NSO can tell you about the current configuration, as well as how you can update the configuration across your network with ease.  

### Reading Device Configuration 
When you did the initial `sync-from` the network NSO retrieved all the current configurations of the devices and stored them in the "CDB".  This is NSO's Configuration Database, and the heart of the "magic" that makes NSO so powerful.  With the CDB, you now have access to the entire network's configuration from a single point through the NSO CLI, web ui, or through the variety of APIs NSO makes available. 

Even more exciting, this configuration isn't just another text file backup, but rather a fully modeled configuration. All of your network devices, even old legacy CLI devices, now have a fully parsed local snapshot of the configuration.  If that didn't **WOW** you, just wait!

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

    > Don't forget you can use TAB completion.  And notice the use of `nx:` in the configuration to indicate the NED or device type we are working with. Also note that the tab completion is **case sensitive** for that first letter. Common mistakes are nx:interface vlan instead of nx:interface Vlan. 

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
In the previous example we looked at managing the configuration of devices, per device.  But what about when you want to configure something on many devices?  There are several ways NSO can help with this, but the simplest way to get started is using Device Templates.  

In this exercise we'll build a simple device template that can be used to configure the DNS server on network devices.  

1. Go ahead and jump into `ncs_cli` like before, and enter `config` mode.  
1. We'll iterate on our device template a couple times, making it more powerful, but for now we'll give a basic template.  

    ```
    devices template SET-DNS-SERVER
    ned-id cisco-nx-cli-5.13
    config
    ip name-server servers 208.67.222.222
    ip name-server servers 208.67.220.220
    ```

1. Let's see what this looks like to NSO. 

    ```
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    devices template SET-DNS-SERVER
     ned-id cisco-nx-cli-5.13
      config
       ip name-server servers [ 208.67.222.222 208.67.220.220 ]
    ```
    </details>

    > One thing to notice here is how NSO takes the input of 2 different name servers and combines them into a list object within the CDB.  

1. The first version of our template only addresses NX-OS devices (based on the `ned-id` block).  We'll circle back to add other network devices in a moment, but let's test this first version. 
1. Commit the new template to NSO. 

    ```
    commit
    ```

1. Jump back to the `top` and let's test out the template. 

    ```
    top
    devices device leaf01-1 apply-template template-name SET-DNS-SERVER
    ```

    <details><summary>Output</summary>

    ```
    apply-template-result {
        device leaf01-1
        result ok
    }
    ```
    </details>

1. Like other configuration changes, this simply stages a configuration to be pushed.  Look at what is staged. 

    ```
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    devices device leaf01-1
     config
      ip name-server 208.67.220.220 208.67.222.222
    ```
    </details>

1. Looks good, but we aren't done yet... let's un-stage these changes.

    ```
    revert
    ```

1. At the prompt, enter `yes` to confirm. 

    ```
    ! EXAMPLE
    All configuration changes will be lost. Proceed? [yes, NO] yes
    ```

1. We're now going to update the template to support IOS and ASA devices as well.  Enter this configuration into NSO. 

    ```
    devices template SET-DNS-SERVER
    ned-id cisco-ios-cli-6.42
    config
    ip name-server name-server-list 208.67.222.222
    ip name-server name-server-list 208.67.220.220
    exit
    exit

    ! 2 exits to return back to the 'config-template-SET-DNS-SERVER' context
    ned-id cisco-asa-cli-6.7
    config
    dns domain-lookup mgmt
    dns server-group DefaultDNS
    name-server 208.67.220.220
    name-server 208.67.222.222
    ```

    > Now our template knows how to configure the DNS Servers on all three of our network types.  

1. Save the changes to the template and return to `top`

    ```
    commit
    top
    ```

1. Now we'll use the `device-groups` that we setup initially to push this change out to the entire network with super ease. 

    ```
    devices device-group ALL apply-template template-name SET-DNS-SERVER
    ```


    <details><summary>Output</summary>

    ```
    apply-template-result {
        device dmz-fw01-1
        result ok
    }
    apply-template-result {
        device dmz-rtr02-1
        result ok
    }
    apply-template-result {
        device dmz-rtr02-2
        result ok
    }
    apply-template-result {
        device dmz-sw01-1
        result ok
    }
    apply-template-result {
        device dmz-sw01-2
        result ok
    }
    apply-template-result {
        device dmz-sw02-1
        result ok
    }
    apply-template-result {
        device dmz-sw02-2
        result ok
    }
    apply-template-result {
        device fw-inside-sw01
        result ok
    }
    apply-template-result {
        device fw-outside-sw01
        result ok
    }
    apply-template-result {
        device fw01
        result ok
    }
    apply-template-result {
        device fw02
        result ok
    }
    apply-template-result {
        device internet-rtr
        result ok
    }
    apply-template-result {
        device leaf01-1
        result ok
    }
    apply-template-result {
        device leaf01-2
        result ok
    }
    apply-template-result {
        device leaf02-1
        result ok
    }
    apply-template-result {
        device leaf02-2
        result ok
    }
    apply-template-result {
        device spine01-1
        result ok
    }
    apply-template-result {
        device spine01-2
        result ok
    }
    apply-template-result {
        device vm-sw01
        result ok
    }
    apply-template-result {
        device vm-sw02
        result ok
    }
    ```
    </details>

    > Pretty awesome right?  One simple command and you've updated every device all at once.  That's AUTOMATION!

1. And let's see what configuration is ready to be sent to the devices.  You can look at `show configuration` if you'd like, but we'll look at the native configuration. 

    ```
    commit dry-run outformat native
    ```

    <details><summary>Output</summary>

    ```
    native {
        device {
            name dmz-rtr02-1
            data ip name-server 208.67.222.222
                ip name-server 208.67.220.220
        }
        device {
            name dmz-rtr02-2
            data ip name-server 208.67.222.222
                ip name-server 208.67.220.220
        }
        device {
            name dmz-sw01-1
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name dmz-sw01-2
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name dmz-sw02-1
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name dmz-sw02-2
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name fw-inside-sw01
            data ip name-server 208.67.222.222
                ip name-server 208.67.220.220
        }
        device {
            name fw-outside-sw01
            data ip name-server 208.67.222.222
                ip name-server 208.67.220.220
        }
        device {
            name fw01
            data dns domain-lookup mgmt
                dns server-group DefaultDNS
                name-server 208.67.220.220
                name-server 208.67.222.222
                exit
        }
        device {
            name fw02
            data dns domain-lookup mgmt
                dns server-group DefaultDNS
                name-server 208.67.220.220
                name-server 208.67.222.222
                exit
        }
        device {
            name internet-rtr
            data ip name-server 208.67.222.222
                ip name-server 208.67.220.220
        }
        device {
            name leaf01-1
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name leaf01-2
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name leaf02-1
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name leaf02-2
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name spine01-1
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name spine01-2
            data ip name-server 208.67.220.220 208.67.222.222
        }
        device {
            name vm-sw01
            data ip name-server 208.67.222.222
                ip name-server 208.67.220.220
        }
        device {
            name vm-sw02
            data ip name-server 208.67.222.222
                ip name-server 208.67.220.220
        }
    }
    ```
    </details>    

1. Finally, push it out to the devices. 

    ```
    commit
    ```

This is just a quick introduction to templates.  Templates have many other uses and features, we'll see more in later exercises.  

## Device Operational State
We'll leave device configuration behind for a little bit and see what NSO offers from an operational state view.  

Much of network automation, particularly the early projects relate not to updating configuration, but rather gathering details about the current state of the network for visibility and troubleshooting.  

Along with modeling out the device configuration, NEDs will often include the ability to read current operational state from the network - similar to running `show` commands on a device.  We'll checkout some of them now.  

1. Go ahead and launch `ncs_cli`, but **don't** go into `config` mode.  

### Platform Info 
Here's one that has saved me all sorts of time - basic platform information.  As part of the initial connection to a device, NSO queries the device for it's model, serial, version, etc.  

1. Take a look at what is gathered.

    ```
    show devices device leaf01-1 platform                                 
    ```

    <details><summary>Output</summary>

    ```
    platform name NX-OS
    platform version 9.2(2)
    platform model "cisco Nexus9000 9000v Chassis "
    platform serial-number 9EMO859VJOO
    ```
    </details>

1. Of if you just want the serial number, you can of course... 

    ```
    show devices device leaf01-1 platform serial-number
    ```

1. And to go even bigger... what if you want **ALL** serial numbers?  

    ```
    show devices device * platform serial-number 
    ```

    <details><summary>Output</summary>

    ```
                    SERIAL       
    NAME             NUMBER       
    ------------------------------
    dmz-fw01-1       9AHFCCB49J1  
    dmz-rtr01-1      97DHMECAG7T  
    dmz-rtr02-1      97DHMECAG7T  
    dmz-rtr02-2      9X83MITDMNZ  
    dmz-sw01-1       9TJ2V92YN1J  
    dmz-sw01-2       9I95731WUFD  
    dmz-sw02-1       9S151GTQ5EX  
    dmz-sw02-2       9SH0ETX3AU3  
    fw-inside-sw01   9YSYIAS6DX5  
    fw-outside-sw01  947MPGL9D9W  
    fw01             9AFK2A2QF29  
    fw02             9ALWBDLW2L6  
    internet-rtr     9AXL4JQRMM6  
    leaf01-1         9EMO859VJOO  
    leaf01-2         97HPIKZ4FFI  
    leaf02-1         9I6F9OKAEOY  
    leaf02-2         9DMNLYGAICX  
    spine01-1        957MQP34EU4  
    spine01-2        9HLJ93K3214  
    vm-sw01          9PF0O7C0LZE  
    vm-sw02          9YKKIHL53I5
    ```
    </details>

    > That wildcard is super handy huh?!?

#### Live Status
Devices have a `live-status` part of the model that indicates details that NSO will read from the device at the time of execution, rather than pulled from the CDB.  

1. Let's check on the current status of the port-channels on a device. 

    ```
    show devices device leaf01-1 live-status port-channel
    ```

    <details><summary>Output</summary>

    ```
                                                                    PORT    
    GROUP  PORT CHANNEL   LAYER  STATUS  TYPE  PRTCL  PORT          STATUS  
    ------------------------------------------------------------------------
    1      port-channel1  S      U       Eth   LACP   Ethernet1/1   P       
                                                    Ethernet1/2   P       
    2      port-channel2  S      U       Eth   LACP   Ethernet1/3   P       
                                                    Ethernet1/4   P       
    3      port-channel3  S      U       Eth   LACP   Ethernet1/11  P  
    ```
    </details>

1. Or maybe the routing table. 

    ```
    show devices device leaf01-1 live-status ip route
    ```

    <details><summary>Partial Output</summary>

    ```
    live-status ip route vrf backdoor
    prefix 172.16.30.0/24
    ucast-nhops 1
    mcast-nhops 0
    attached    true
    path 172.16.30.2
    uptime     P4DT2H41M12S
    ifname     Vlan3001
    pref       0
    metric     0
    clientname direct
    ubest      true
    prefix 172.16.30.1/32
    ucast-nhops 1
    mcast-nhops 0
    attached    true
    path 172.16.30.1
    uptime     P4DT2H40M50S
    ifname     Vlan3001
    pref       0
    metric     0
    clientname hsrp
    ubest      true
    prefix 172.16.30.2/32
    ucast-nhops 1
    mcast-nhops 0
    attached    true
    path 172.16.30.2
    uptime     P4DT2H41M12S
    ifname     Vlan3001
    pref       0
    metric     0
    clientname local
    ubest      true
    live-status ip route vrf default
    live-status ip route vrf dmz
    prefix 0.0.0.0/0
    ucast-nhops 1
    mcast-nhops 0
    attached    false
    path 10.225.250.4
    uptime     P4DT2H39M6S
    pref       1
    metric     0
    clientname static
    ubest      true
    prefix 10.225.1.0/24
    ucast-nhops 1
    mcast-nhops 0
    attached    false
    path 10.225.250.36
    uptime     P4DT2H38M58S
    ifname     Vlan100
    pref       110
    metric     90
    clientname ospf-1
    ubest      true
    .
    .
    ```
    </details>

    * Uh, oh... the default formating isn't super friendly to the human reader.  Luckily NSO can force a table format for output.  

    ```
    show devices device leaf01-1 live-status ip route | tab
    ```

    <details><summary>Partial Output</summary>

    ```
                                UCAST  MCAST                                                                                    
    NAME        IPPREFIX          NHOPS  NHOPS  ATTACHED  IPNEXTHOP      UPTIME        IFNAME    PREF  METRIC  CLIENTNAME  UBEST  
    ------------------------------------------------------------------------------------------------------------------------------
    backdoor    172.16.30.0/24    1      0      true      172.16.30.2    P4DT2H44M33S  Vlan3001  0     0       direct      true   
                172.16.30.1/32    1      0      true      172.16.30.1    P4DT2H44M11S  Vlan3001  0     0       hsrp        true   
                172.16.30.2/32    1      0      true      172.16.30.2    P4DT2H44M33S  Vlan3001  0     0       local       true   
    default                                                                                                                       
    dmz         0.0.0.0/0         1      0      false     10.225.250.4   P4DT2H42M27S  -         1     0       static      true   
                10.225.1.0/24     1      0      false     10.225.250.36  P4DT2H42M19S  Vlan100   110   90      ospf-1      true   
                10.225.10.0/24    1      0      false     10.225.250.36  P4DT2H42M19S  Vlan100   110   90      ospf-1      true   
                10.225.11.0/24    1      0      false     10.225.250.36  P4DT2H42M19S  Vlan100   110   90      ospf-1      true   
                10.225.17.0/24    1      0      true      10.225.17.2    P4DT2H44M49S  Vlan201   0     0       direct      true   
                10.225.17.1/32    1      0      true      10.225.17.1    P4DT2H44M26S  Vlan201   0     0       hsrp        true   
                10.225.17.2/32    1      0      true      10.225.17.2    P4DT2H44M49S  Vlan201   0     0       local       true   
                10.225.18.0/24    1      0      false     10.225.250.36  P4DT2H42M19S  Vlan100   110   90      ospf-1      true   
                10.225.19.0/24    1      0      false     10.225.250.36  P4DT2H42M19S  Vlan100   110   90      ospf-1      true   
                10.225.2.0/24     1      0      false     10.225.250.36  P4DT2H42M19S  Vlan100   110   90      ospf-1      true   
                10.225.250.0/29   1      0      false     10.225.250.36  P4DT2H42M27S  Vlan100   110   50      ospf-1      true   
                10.225.250.16/29  1      0      false     10.225.250.36  P4DT2H42M27S  Vlan100   110   51      ospf-1      true   
                10.225.250.24/29  1      0      false     10.225.250.36  P4DT2H42M27S  Vlan100   110   51      ospf-1      true   
                10.225.250.32/29  1      0      true      10.225.250.34  P4DT2H45M1S   Vlan100   0     0       direct      true   
                10.225.250.33/32  1      0      true      10.225.250.33  P4DT2H44M39S  Vlan100   0     0       hsrp        true   
                10.225.250.34/32  1      0      true      10.225.250.34  P4DT2H45M1S   Vlan100   0     0       local       true   
                10.225.250.48/29  1      0      false     10.225.250.36  P4DT2H42M27S  Vlan100   110   50      ospf-1      true   
                10.225.250.8/29   1      0      false     10.225.250.36  P4DT2H42M27S  Vlan100   110   50      ospf-1      true   
                10.225.49.0/24    1      0      false     10.225.250.36  P4DT2H42M19S  Vlan100   110   90      ospf-1      true   
                10.225.64.0/24    1      0      false     10.225.250.36  P4DT2H42M27S  Vlan100   110   20      ospf-1      true   
                10.225.64.0/27    1      0      false     10.225.250.36  P4DT2H42M27S  Vlan100   110   20      ospf-1      true   
                10.225.9.0/24     1      0      true      10.225.9.2     P4DT2H44M58S  Vlan101   0     0       direct      true   

    ```
    </details>  

There is a lot of information available through `live-status` feel free to poke around and explore. 

## Live Status and Arbitrary Commands
NSO is great, but occasionally the NED may not support the specific configuration bit, or operational data that you're looking for.  Now when this happens for a customer, they can easily open a support ticket with the Cisco NSO team explaining what is missing, and the NEDs can be updated to support what is needed. These updates often only take a few days to be vetted, tested, and implemented by Cisco and updated NEDs made available to the customer. 

> Note: Once you become familiar with the more advanced topics around NSO development, you could even build your own packages and code to add these features yourself.

However, for immediate needs, `live-status` supports sending arbitrary commands to a network device and returning the result.  Let's see a quick example of this in action.  

1. Suppose you needed to check the license usage on all your Nexus switches.  This isn't data that the NED currently supports (though I probably should open a request to have it added), so we'll use `live-status` to gather the info. 

1. First, let's check the status on a single device. 

    ```
    devices device leaf01-1 live-status exec show license usage
    ```

    <details><summary>Output</summary>

    ```
    result 
    Feature                      Ins  Lic   Status Expiry Date Comments
                                    Count
    --------------------------------------------------------------------------------
    N9K_LIC_1G                    No    -   Unused             -
    VPN_FABRIC                    No    -   Unused             -
    NXOS_OE_PKG                   No    -   Unused             -
    FCOE_NPV_PKG                  No    -   Unused             -
    SECURITY_PKG                  No    0   Unused             -
    N9K_UPG_EX_10G                No    -   Unused             -
    TP_SERVICES_PKG               No    -   Unused             -
    NXOS_ADVANTAGE_GF             No    -   Unused             -
    NXOS_ADVANTAGE_M4             No    -   Unused             -
    NXOS_ADVANTAGE_XF             No    -   Unused             -
    NXOS_ESSENTIALS_GF            No    -   Unused             -
    NXOS_ESSENTIALS_M4            No    -   Unused             -
    NXOS_ESSENTIALS_XF            No    -   Unused             -
    SAN_ENTERPRISE_PKG            No    -   Unused             -
    PORT_ACTIVATION_PKG           No    0   Unused             -
    NETWORK_SERVICES_PKG          No    -   Unused             -
    NXOS_ADVANTAGE_M8-16          No    -   Unused             -
    NXOS_ESSENTIALS_M8-16         No    -   Unused             -
    FC_PORT_ACTIVATION_PKG        No    0   Unused             -
    LAN_ENTERPRISE_SERVICES_PKG   No    -   Unused             -
    --------------------------------------------------------------------------------
    ```
    </details>

    > Notice that this isn't a `show` command.

1. Now we'll use wild-card commands to run the same against all the `leaf` switches.  

    > Note: You can't use `live-status` with `device-groups` because a group of device could leverage different NEDs. 

    ```
    devices device leaf* live-status exec show license usage
    ```

    <details><summary>Partial Output</summary>

    ```
    devices device leaf01-1 live-status exec show
        result 
    Feature                      Ins  Lic   Status Expiry Date Comments
                                    Count
    --------------------------------------------------------------------------------
    N9K_LIC_1G                    No    -   Unused             -
    VPN_FABRIC                    No    -   Unused             -
    NXOS_OE_PKG                   No    -   Unused             -
    FCOE_NPV_PKG                  No    -   Unused             -
    SECURITY_PKG                  No    0   Unused             -
    N9K_UPG_EX_10G                No    -   Unused             -
    TP_SERVICES_PKG               No    -   Unused             -
    NXOS_ADVANTAGE_GF             No    -   Unused             -
    NXOS_ADVANTAGE_M4             No    -   Unused             -
    NXOS_ADVANTAGE_XF             No    -   Unused             -
    NXOS_ESSENTIALS_GF            No    -   Unused             -
    NXOS_ESSENTIALS_M4            No    -   Unused             -
    NXOS_ESSENTIALS_XF            No    -   Unused             -
    SAN_ENTERPRISE_PKG            No    -   Unused             -
    PORT_ACTIVATION_PKG           No    0   Unused             -
    NETWORK_SERVICES_PKG          No    -   Unused             -
    NXOS_ADVANTAGE_M8-16          No    -   Unused             -
    NXOS_ESSENTIALS_M8-16         No    -   Unused             -
    FC_PORT_ACTIVATION_PKG        No    0   Unused             -
    LAN_ENTERPRISE_SERVICES_PKG   No    -   Unused             -
    --------------------------------------------------------------------------------

    devices device leaf01-2 live-status exec show
        result 
    Feature                      Ins  Lic   Status Expiry Date Comments
                                    Count
    --------------------------------------------------------------------------------
    N9K_LIC_1G                    No    -   Unused             -
    VPN_FABRIC                    No    -   Unused             -
    NXOS_OE_PKG                   No    -   Unused             -
    FCOE_NPV_PKG                  No    -   Unused             -
    SECURITY_PKG                  No    0   Unused             -
    N9K_UPG_EX_10G                No    -   Unused             -
    TP_SERVICES_PKG               No    -   Unused             -
    NXOS_ADVANTAGE_GF             No    -   Unused             -
    NXOS_ADVANTAGE_M4             No    -   Unused             -
    NXOS_ADVANTAGE_XF             No    -   Unused             -
    NXOS_ESSENTIALS_GF            No    -   Unused             -
    NXOS_ESSENTIALS_M4            No    -   Unused             -
    NXOS_ESSENTIALS_XF            No    -   Unused             -
    SAN_ENTERPRISE_PKG            No    -   Unused             -
    PORT_ACTIVATION_PKG           No    0   Unused             -
    NETWORK_SERVICES_PKG          No    -   Unused             -
    NXOS_ADVANTAGE_M8-16          No    -   Unused             -
    NXOS_ESSENTIALS_M8-16         No    -   Unused             -
    FC_PORT_ACTIVATION_PKG        No    0   Unused             -
    LAN_ENTERPRISE_SERVICES_PKG   No    -   Unused             -
    --------------------------------------------------------------------------------

    devices device leaf02-1 live-status exec show
        result 
    Feature                      Ins  Lic   Status Expiry Date Comments
                                    Count
    --------------------------------------------------------------------------------
    N9K_LIC_1G                    No    -   Unused             -
    VPN_FABRIC                    No    -   Unused             -
    NXOS_OE_PKG                   No    -   Unused             -
    FCOE_NPV_PKG                  No    -   Unused             -
    SECURITY_PKG                  No    0   Unused             -
    N9K_UPG_EX_10G                No    -   Unused             -
    TP_SERVICES_PKG               No    -   Unused             -
    NXOS_ADVANTAGE_GF             No    -   Unused             -
    NXOS_ADVANTAGE_M4             No    -   Unused             -
    NXOS_ADVANTAGE_XF             No    -   Unused             -
    NXOS_ESSENTIALS_GF            No    -   Unused             -
    NXOS_ESSENTIALS_M4            No    -   Unused             -
    NXOS_ESSENTIALS_XF            No    -   Unused             -
    SAN_ENTERPRISE_PKG            No    -   Unused             -
    PORT_ACTIVATION_PKG           No    0   Unused             -
    NETWORK_SERVICES_PKG          No    -   Unused             -
    NXOS_ADVANTAGE_M8-16          No    -   Unused             -
    NXOS_ESSENTIALS_M8-16         No    -   Unused             -
    FC_PORT_ACTIVATION_PKG        No    0   Unused             -
    LAN_ENTERPRISE_SERVICES_PKG   No    -   Unused             -
    --------------------------------------------------------------------------------

    devices device leaf02-2 live-status exec show
        result 
    Feature                      Ins  Lic   Status Expiry Date Comments
                                    Count
    --------------------------------------------------------------------------------
    N9K_LIC_1G                    No    -   Unused             -
    VPN_FABRIC                    No    -   Unused             -
    NXOS_OE_PKG                   No    -   Unused             -
    .
    .
    ```
    </details>

1. Many times you need to gather data like this for later processing, or to share with someone.  For any command, NSO allows you to `save` the output to a file. 

    ```
    devices device leaf* live-status exec show license usage | save leaf-license-usage.txt
    ```

1. If you exit from NSO, you'll find a new file in your current directory with this data.  

    ```bash
    ll leaf-license-usage.txt 
    -rw-r--r--  1 hapresto  staff   6.2K Dec 19 15:22 leaf-license-usage.txt
    ```
