# Minimum Viable Use Case: Update SNMP Configuration Across the Network
My go-to example for a first network automation project has long been the updating of SNMP community strings because it is commonly used and has a limited blast radius if something goes wrong as you are learning.  Let's see how NSO can make it super easy. 

## Use Case Description 
Our goal is to update the SNMP community string configuration on all our network devices.  This includes: 

* NX-OS switches
* IOS switches and routers 
* ASA firewalls

## Cisco NSO Features Used 
* Device Templates with Variables and Tags
* Device Groups 

## Building the Use Case 
### Step 1: Create the Template 
1. Log into NSO with `ncs_cli -C -u admin` and enter `config` mode. 

1. Let's setup the device template that covers the three device-types in question.  

    ```
    devices template UPDATE-SNMP-COMMUNITY
    ned-id cisco-ios-cli-6.40
    config
    snmp-server community {$READ_ONLY_COMMUNITY} RO
    snmp-server community {$READ_WRITE_COMMUNITY} RW
    exit
    exit
    exit

    ned-id cisco-nx-cli-5.13
    config
    snmp-server community {$READ_ONLY_COMMUNITY} group network-operator
    snmp-server community {$READ_WRITE_COMMUNITY} group network-admin
    exit
    exit
    exit

    ned-id cisco-asa-cli-6.7
    config
    snmp-server community secret {$READ_ONLY_COMMUNITY}
    snmp-server enable-conf enable true
    exit
    exit
    ```

    * NSO device templates can use variables for values to make them more dynamic and re-usable. The variable syntax is {$VAR_NAME}, though the variable name does not have to be in all caps, we do that to make it more readable. 
    * The syntax of the template configuration may differ slightly from what a native device CLI configuration is.  That is because the template is a bit more verbose of a representation of the NED data model.  

1. Return to `top` and checkout the configuration with `show configuration`
1. Go ahead and save this new template. 

    ```
    commit
    ```

### Step 2: Testing the Template 
You always need to test your automation before pushing it out to the whole network. Let's verify that it works as expected on our three platforms.  

1. First let's see what SNMP configuration exists on our test platforms. We'll do this from the NSO CLI "enable" mode. 
    > Note: From `config` mode you can use `show full-configuration` to look at the device configuration.  This will display both the `running-config` as well as any staged configuration that NSO is planning on sending to the device.  

    ```
    ! "end" will exit config mode
    end
    show running-config devices device dmz-rtr02-1 config snmp-server
    show running-config devices device leaf01-1 config snmp-server
    show running-config devices device fw01 config snmp-server
    ```

    <details><summary>Output</summary>

    ```
    admin@ncs# show running-config devices device dmz-rtr02-2 config snmp-server
    % No entries found.
    admin@ncs# show running-config devices device leaf01-1 config snmp-server
    % No entries found.
    admin@ncs# show running-config devices device fw01 config snmp-server
    ! 2 exits to return back to the 'config-template-SET-DNS-SERVER' context
    devices device fw01
     config
      no snmp-server contact
      no snmp-server location
    ```
    </details>

1. Next up, let's test it on these three devices.  
    > Note: The syntax to input the variable values for a particular device is `devices device DEVICE-NAME apply-template template-name TEMPLATE-NAME variable { name VARIABLE-NAME1 value 'value-in-template1' } variable { name VARIABLE-NAME2 value 'value-in-template2' }`. 
    ```
    config 

    devices device dmz-rtr02-1 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_1' } variable { name READ_WRITE_COMMUNITY value 'WRITE_1' }

    devices device leaf01-1 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_1' } variable { name READ_WRITE_COMMUNITY value 'WRITE_1' }

    devices device fw01 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_1' } variable { name READ_WRITE_COMMUNITY value 'WRITE_1' }
    ```

    > Note: String values for variables need to be surrounded by single quotes.


    <details><summary>Output</summary>

    ```
    admin@ncs(config)# devices device dmz-rtr02-1 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_1' } variable { name READ_WRITE_COMMUNITY value 'WRITE_1' }
    apply-template-result {
        device dmz-rtr02-1
        result ok
    }
    admin@ncs(config)# devices device leaf01-1 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_1' } variable { name READ_WRITE_COMMUNITY value 'WRITE_1' }
    apply-template-result {
        device leaf01-1
        result ok
    }
    admin@ncs(config)# devices device fw01 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_1' } variable { name READ_WRITE_COMMUNITY value 'WRITE_1' }
    apply-template-result {
        device fw01
        result ok
    }
    ```
    </details>

1. Now that NSO has the template applied with variable values defined, it can compare the intended configuration from the template to the actual configuration stored in the NSO CDB, and calculate what configuration needs to be sent to make the device be in compliance. Since there is no pre-existing SNMP configuration, all the commands would be sent, but in a brownfield network device, it would send only the minimal amount of commands needed. 

To view the resulting pending configuration to be applied from the templates use the following command:  

    ```
    show configuration 
    ```

    <details><summary>Output</summary>

    ```
    devices device dmz-rtr02-1
    config
    snmp-server community READ_1 RO
    snmp-server community WRITE_1 RW
    !
    !
    devices device fw01
    config
    snmp-server community READ_1
    snmp-server enable
    !
    !
    devices device leaf01-1
    config
    snmp-server community READ_1 group network-operator
    snmp-server community WRITE_1 group network-admin
    ```
    </details>

    ```
    commit dry-run outformat native
    ```

    <details><summary>Output</summary>

    ```
    native {
        device {
            name dmz-rtr02-1
            data snmp-server community READ_1 RO
                snmp-server community WRITE_1 RW
        }
        device {
            name fw01
            data snmp-server community READ_1
                snmp-server enable
        }
        device {
            name leaf01-1
            data snmp-server community READ_1 group network-operator
                snmp-server community WRITE_1 group network-admin
        }
    }
    ```
    </details>

1. It all looks good, let's commit it, so NSO will apply the pending configuration to the devices. 

    ```
    commit
    ```


### Step 3: Deeper Testing and a Reference to a Classic Dinosaur Movie

1. Now the goal of this use case is to replace/update the community string.  Let's see what happens when we run the templates again to change the string. 

    ```
    devices device dmz-rtr02-1 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_2' } variable { name READ_WRITE_COMMUNITY value 'WRITE_2' }

    devices device leaf01-1 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_2' } variable { name READ_WRITE_COMMUNITY value 'WRITE_2' }

    devices device fw01 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_2' } variable { name READ_WRITE_COMMUNITY value 'WRITE_2' }
    ```

> **IMPORTANT PART** - Let's spend a bit of time exploring what's going on to uncover a big challenge with many network automation projects.  Something I like to call the "Unexpected Raptors Problem". 

1. Let's look at our normal verification before committing.  

    ```
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    devices device dmz-rtr02-1
    config
    snmp-server community READ_2 RO
    snmp-server community WRITE_2 RW
    !
    !
    devices device fw01
    config
    snmp-server community READ_2
    !
    !
    devices device leaf01-1
    config
    snmp-server community READ_2 group network-operator
    snmp-server community WRITE_2 group network-admin
    ```
    </details>

1. This looks like exactly what we'd want.  But let's dig a bit deeper and look at the `full-configuration` for these devices (for just the `snmp-server` part of the device). 

    ```
    show full-configuration devices device dmz-rtr02-1 config snmp-server
    show full-configuration devices device leaf01-1 config snmp-server
    show full-configuration devices device fw01 config snmp-server
    ```

    <details><summary>Output</summary>

    ```
    admin@ncs(config)# show full-configuration devices device dmz-rtr02-1 config snmp-server
    ! 2 exits to return back to the 'config-template-SET-DNS-SERVER' context
    devices device dmz-rtr02-1
    config
    snmp-server community READ_1 RO
    snmp-server community READ_2 RO
    snmp-server community WRITE_1 RW
    snmp-server community WRITE_2 RW
    !
    !
    admin@ncs(config)# show full-configuration devices device leaf01-1 config snmp-server
    ! 2 exits to return back to the 'config-template-SET-DNS-SERVER' context
    devices device leaf01-1
    config
    snmp-server community READ_1 group network-operator
    snmp-server community READ_2 group network-operator
    snmp-server community WRITE_1 group network-admin
    snmp-server community WRITE_2 group network-admin
    !
    !
    admin@ncs(config)# show full-configuration devices device fw01 config snmp-server
    ! 2 exits to return back to the 'config-template-SET-DNS-SERVER' context
    devices device fw01
    config
    snmp-server community READ_2
    no snmp-server contact
    snmp-server enable
    no snmp-server location
    ```
    </details>

    > Well look at that... the IOS and NX devices have both community strings, while the ASA only has the new one.  This is because communities on IOS and NX are lists, the devices support having many configured simultaneously, while ASA only supports one. This is why it is important to test your configuration before deploying because different network OS platforms behave differently, even for similar configuration changes. 

1. Go ahead and commit this change, and we'll tackle fixing this in our next step. 

    ```
    commit
    ```

### Step 4: Getting Rid of the Extra Dinosaurs with Tags!
The default behavior of NSO templates is to `merge` data into the current configuration.  This is often what you want to happen, but not always.  

In these cases, you can tell NSO to take one of the following alternative actions: 

* `merge`: Merge with a node if it exists, otherwise create the node. *This is the default operation if no operation is explicitly set.*
* `replace`: Replace a node if it exists, otherwise create the node.
* `create`: Creates a node. The node can not already exist.
* `nocreate`: Merge with a node if it exists. If it does not exist, it will not be created.

1. Start out by looking at the full-configuration of the template we are using. 

    ```
    show full-configuration devices template UPDATE-SNMP-COMMUNITY 
    ```

    ```
    devices template UPDATE-SNMP-COMMUNITY
     ned-id cisco-ios-cli-6.40
      config
       snmp-server community {$READ_ONLY_COMMUNITY}
         RO
       !
       snmp-server community {$READ_WRITE_COMMUNITY}
         RW
       !
      !
     !
     ned-id cisco-nx-cli-5.13
      config
       snmp-server community {$READ_ONLY_COMMUNITY}
         group network-operator
       !
       snmp-server community {$READ_WRITE_COMMUNITY}
         group network-admin
       !
      !
     !
     ned-id cisco-asa-cli-6.7
      config
       snmp-server community secret {$READ_ONLY_COMMUNITY}
       snmp-server enable-conf enable true
    ```

1. Within NSO templates, you can change the behavior of templates being applied by  leveraging `tags` attached to parts of the configuration. These can be tactical, for just one little portion of the configuration, or be applied to the whole template.  A `tag` will apply to all sub-nodes from the point it is added,.  The meaning of that makes far more sense when you look at the configuration in the XML format, since it is a hierarchical data format.  

    ```
    show full-configuration devices template UPDATE-SNMP-COMMUNITY | display xml
    ```

    <details><summary>Full Output</summary>

    ```xml
    <config xmlns="http://tail-f.com/ns/config/1.0">
    <devices xmlns="http://tail-f.com/ns/ncs">
    <template>
        <name>UPDATE-SNMP-COMMUNITY</name>
        <ned-id>
        <id xmlns:cisco-ios-cli-6.40="http://tail-f.com/ns/ned-id/cisco-ios-cli-6.40">cisco-ios-cli-6.40:cisco-ios-cli-6.40</id>
        <config>
            <snmp-server xmlns="urn:ios">
            <community>
                <name>{$READ_ONLY_COMMUNITY}</name>
                <RO/>
            </community>
            <community>
                <name>{$READ_WRITE_COMMUNITY}</name>
                <RW/>
            </community>
            </snmp-server>
        </config>
        </ned-id>
        <ned-id>
        <id xmlns:cisco-nx-cli-5.13="http://tail-f.com/ns/ned-id/cisco-nx-cli-5.13">cisco-nx-cli-5.13:cisco-nx-cli-5.13</id>
        <config>
            <snmp-server xmlns="http://tail-f.com/ned/cisco-nx">
            <community>
                <snmp-community-string>{$READ_ONLY_COMMUNITY}</snmp-community-string>
                <group>network-operator</group>
            </community>
            <community>
                <snmp-community-string>{$READ_WRITE_COMMUNITY}</snmp-community-string>
                <group>network-admin</group>
            </community>
            </snmp-server>
        </config>
        </ned-id>
        <ned-id>
        <id xmlns:cisco-asa-cli-6.7="http://tail-f.com/ns/ned-id/cisco-asa-cli-6.7">cisco-asa-cli-6.7:cisco-asa-cli-6.7</id>
        <config>
            <snmp-server xmlns="http://cisco.com/ned/asa">
            <community>
                <secret>{$READ_ONLY_COMMUNITY}</secret>
            </community>
            <enable-conf>
                <enable>true</enable>
            </enable-conf>
            </snmp-server>
        </config>
        </ned-id>
    </template>
    </devices>
    </config>
    ```
    </details>
   
    > Note: XML is a hierarchical language, with each opening tag (`<template>`) needing a closing tag (`</template>`). Indentation helps see the hierarchical relationships of XML, but the indentation does not affect the processing of XML.  

1. What we want to do is `replace` everything within the `<snmp-server>` parts of the configuration.  Here's an example. 

    ```text
        <config>
          <snmp-server xmlns="urn:ios">                 <--|
            <community>                                    |
                <name>{$READ_ONLY_COMMUNITY}</name>        |
                <RO/>                                      |
            </community>                                   |
            <community>                                    |
                <name>{$READ_WRITE_COMMUNITY}</name>       |
                <RW/>                                      |
            </community>                                   |
          </snmp-server>                                <--|
        </config>
    ```

    > We want to replace **ALL** the communities within the `<snmp-server>` block.

1. To do that, we'll `add` the `replace` tag to each of the `snmp-server` elements of the configuration in the template. 

    ```
    devices template UPDATE-SNMP-COMMUNITY
    ned-id cisco-ios-cli-6.40 
    config
    tag add snmp-server replace

    ned-id cisco-nx-cli-5.13 
    config
    tag add snmp-server replace

    ned-id cisco-asa-cli-6.7 
    config
    tag add snmp-server replace
    exit
    exit
    ```

1. Take a look at the `full-configuration` now to see what it looks like. 

    ```
    top
    show full-configuration devices template UPDATE-SNMP-COMMUNITY
    ```

    <details><summary>Output</summary>

    ```
    devices template UPDATE-SNMP-COMMUNITY
     ned-id cisco-ios-cli-6.40
      config
       ! Tags: replace (/devices/template{UPDATE-SNMP-COMMUNITY}/ned-id{cisco-ios-cli-6.40:cisco-ios-cli-6.40}/config/ios:snmp-server)
       snmp-server community {$READ_ONLY_COMMUNITY}
         RO
       !
       ! Tags: replace (/devices/template{UPDATE-SNMP-COMMUNITY}/ned-id{cisco-ios-cli-6.40:cisco-ios-cli-6.40}/config/ios:snmp-server)
       snmp-server community {$READ_WRITE_COMMUNITY}
         RW
       !
      !
     !
     ned-id cisco-nx-cli-5.13
      config
       ! Tags: replace (/devices/template{UPDATE-SNMP-COMMUNITY}/ned-id{cisco-nx-cli-5.13:cisco-nx-cli-5.13}/config/nx:snmp-server)
       snmp-server community {$READ_ONLY_COMMUNITY}
         group network-operator
       !
       ! Tags: replace (/devices/template{UPDATE-SNMP-COMMUNITY}/ned-id{cisco-nx-cli-5.13:cisco-nx-cli-5.13}/config/nx:snmp-server)
       snmp-server community {$READ_WRITE_COMMUNITY}
         group network-admin
       !
      !
     !
     ned-id cisco-asa-cli-6.7
      config
       ! Tags: replace (/devices/template{UPDATE-SNMP-COMMUNITY}/ned-id{cisco-asa-cli-6.7:cisco-asa-cli-6.7}/config/asa:snmp-server)
       snmp-server community secret {$READ_ONLY_COMMUNITY}
       ! Tags: replace (/devices/template{UPDATE-SNMP-COMMUNITY}/ned-id{cisco-asa-cli-6.7:cisco-asa-cli-6.7}/config/asa:snmp-server)
       snmp-server enable-conf enable true
    ```

    </details>

    > The tags make the display in CLI format a bit messy, but you can see where the new `Tags: replace` have been added.  What is shown is the XPATH to the point for the relevant tag.  XPATH is a navigation structure for XML. 

1. If you look at the XML version of the config, the placement of the tags is a bit easier to see and understand how they work. In the following output look for the XML meta-data tag `tags=" replace "`. This indicates any existing device configuration below this level will be replaced when the template is applied, rather than the default action of merging configuration. 

    ```
    show full-configuration devices template UPDATE-SNMP-COMMUNITY | display xml
    ```

    <details><summary>Full Output</summary>

    ```xml
    <config xmlns="http://tail-f.com/ns/config/1.0">
    <devices xmlns="http://tail-f.com/ns/ncs">
    <template>
        <name>UPDATE-SNMP-COMMUNITY</name>
        <ned-id>
        <id xmlns:cisco-ios-cli-6.40="http://tail-f.com/ns/ned-id/cisco-ios-cli-6.40">cisco-ios-cli-6.40:cisco-ios-cli-6.40</id>
        <config>
            <snmp-server xmlns="urn:ios" tags=" replace ">  <!-- Notice TAG -->
            <community>
                <name>{$READ_ONLY_COMMUNITY}</name>
                <RO/>
            </community>
            <community>
                <name>{$READ_WRITE_COMMUNITY}</name>
                <RW/>
            </community>
            </snmp-server>
        </config>
        </ned-id>
        <ned-id>
        <id xmlns:cisco-nx-cli-5.13="http://tail-f.com/ns/ned-id/cisco-nx-cli-5.13">cisco-nx-cli-5.13:cisco-nx-cli-5.13</id>
        <config>
            <snmp-server xmlns="http://tail-f.com/ned/cisco-nx" tags=" replace ">  <!-- Notice TAG -->
            <community>
                <snmp-community-string>{$READ_ONLY_COMMUNITY}</snmp-community-string>
                <group>network-operator</group>
            </community>
            <community>
                <snmp-community-string>{$READ_WRITE_COMMUNITY}</snmp-community-string>
                <group>network-admin</group>
            </community>
            </snmp-server>
        </config>
        </ned-id>
        <ned-id>
        <id xmlns:cisco-asa-cli-6.7="http://tail-f.com/ns/ned-id/cisco-asa-cli-6.7">cisco-asa-cli-6.7:cisco-asa-cli-6.7</id>
        <config>
            <snmp-server xmlns="http://cisco.com/ned/asa" tags=" replace ">  <!-- Notice TAG -->
            <community>
                <secret>{$READ_ONLY_COMMUNITY}</secret>
            </community>
            <enable-conf>
                <enable>true</enable>
            </enable-conf>
            </snmp-server>
        </config>
        </ned-id>
    </template>
    </devices>
    </config>
    ```
    </details>

1. Save these changes. 

    ```
    commit
    ```

### Step 5: Another test

1. Now let's test and see if the `tags` worked as expected.  

    ```
    devices device dmz-rtr02-1 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_NO_RAPTOR' } variable { name READ_WRITE_COMMUNITY value 'WRITE_NO_RAPTOR' }

    devices device leaf01-1 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_NO_RAPTOR' } variable { name READ_WRITE_COMMUNITY value 'WRITE_NO_RAPTOR' }

    devices device fw01 apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'READ_NO_RAPTOR' } variable { name READ_WRITE_COMMUNITY value 'WRITE_NO_RAPTOR' }
    ```

1. And look at the resulting configuration. 

    ```
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    devices device dmz-rtr02-1
    config
    no snmp-server community READ_1 RO
    no snmp-server community READ_2 RO
    no snmp-server community WRITE_1 RW
    no snmp-server community WRITE_2 RW
    snmp-server community READ_NO_RAPTOR RO
    snmp-server community WRITE_NO_RAPTOR RW
    !
    !
    devices device fw01
    config
    snmp-server community READ_NO_RAPTOR
    !
    !
    devices device leaf01-1
    config
    no snmp-server community READ_1 group network-operator
    no snmp-server community READ_2 group network-operator
    no snmp-server community WRITE_1 group network-admin
    no snmp-server community WRITE_2 group network-admin
    snmp-server community READ_NO_RAPTOR group network-operator
    snmp-server community WRITE_NO_RAPTOR group network-admin
    ```
    </details>

    > Perfect.. you can see that NSO will remove the old raptors and just keep our good communities. 

## Executing the Use Case 
With our template built and tested, let's put it to work across the network. Rather than applying variable inputs per device, you can also apply templates to a group of devices. 

1. We want to setup SNMP across the entire network, so we'll use the `ALL` group we setup. This would apply the appropriate template per device type, with the same variable inputs, across all the devices in the device-group.   

    ```
    devices device-group ALL apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'INITIAL_READ' } variable { name READ_WRITE_COMMUNITY value 'INITIAL_WRITE' }
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

1. And now verify the pending configuration to be sent to the network devices. 


    > Note: For devices which already have SNMP configuration, like `dmz-rtr02-1`, NSO applies the template to replace the configuration, removing existing SNMP lines. For devices which do not have any configuration to replace, no additional commands are sent.

    ```
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    devices device dmz-fw01-1
    config
    snmp-server community INITIAL_READ
    snmp-server enable
    !
    !
    devices device dmz-rtr02-1
    config
    no snmp-server community READ_1 RO
    no snmp-server community READ_2 RO
    no snmp-server community WRITE_1 RW
    no snmp-server community WRITE_2 RW
    snmp-server community INITIAL_READ RO
    snmp-server community INITIAL_WRITE RW
    !
    !
    devices device dmz-rtr02-2
    config
    snmp-server community INITIAL_READ RO
    snmp-server community INITIAL_WRITE RW
    !
    !
    devices device dmz-sw01-1
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device dmz-sw01-2
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device dmz-sw02-1
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device dmz-sw02-2
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device fw-inside-sw01
    config
    snmp-server community INITIAL_READ RO
    snmp-server community INITIAL_WRITE RW
    !
    !
    devices device fw-outside-sw01
    config
    snmp-server community INITIAL_READ RO
    snmp-server community INITIAL_WRITE RW
    !
    !
    devices device fw01
    config
    snmp-server community INITIAL_READ
    !
    !
    devices device fw02
    config
    snmp-server community INITIAL_READ
    snmp-server enable
    !
    !
    devices device internet-rtr
    config
    snmp-server community INITIAL_READ RO
    snmp-server community INITIAL_WRITE RW
    !
    !
    devices device leaf01-1
    config
    no snmp-server community READ_1 group network-operator
    no snmp-server community READ_2 group network-operator
    no snmp-server community WRITE_1 group network-admin
    no snmp-server community WRITE_2 group network-admin
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device leaf01-2
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device leaf02-1
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device leaf02-2
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device spine01-1
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device spine01-2
    config
    snmp-server community INITIAL_READ group network-operator
    snmp-server community INITIAL_WRITE group network-admin
    !
    !
    devices device vm-sw01
    config
    snmp-server community INITIAL_READ RO
    snmp-server community INITIAL_WRITE RW
    !
    !
    devices device vm-sw02
    config
    snmp-server community INITIAL_READ RO
    snmp-server community INITIAL_WRITE RW
    ```
    </details>

1. Go ahead and commit the change. 

    ```
    commit
    ```

1. And to make sure there aren't any raptors...we test with a new set of inputs to make sure the templates behave as expected. 

    ```
    devices device-group ALL apply-template template-name UPDATE-SNMP-COMMUNITY variable { name READ_ONLY_COMMUNITY value 'NEW_READ' } variable { name READ_WRITE_COMMUNITY value 'NEW_WRITE' }
    ```

1. Verify... 

    ```
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    devices device dmz-fw01-1
    config
    snmp-server community NEW_READ
    !
    !
    devices device dmz-rtr02-1
    config
    no snmp-server community INITIAL_READ RO
    no snmp-server community INITIAL_WRITE RW
    snmp-server community NEW_READ RO
    snmp-server community NEW_WRITE RW
    !
    !
    devices device dmz-rtr02-2
    config
    no snmp-server community INITIAL_READ RO
    no snmp-server community INITIAL_WRITE RW
    snmp-server community NEW_READ RO
    snmp-server community NEW_WRITE RW
    !
    !
    devices device dmz-sw01-1
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device dmz-sw01-2
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device dmz-sw02-1
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device dmz-sw02-2
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device fw-inside-sw01
    config
    no snmp-server community INITIAL_READ RO
    no snmp-server community INITIAL_WRITE RW
    snmp-server community NEW_READ RO
    snmp-server community NEW_WRITE RW
    !
    !
    devices device fw-outside-sw01
    config
    no snmp-server community INITIAL_READ RO
    no snmp-server community INITIAL_WRITE RW
    snmp-server community NEW_READ RO
    snmp-server community NEW_WRITE RW
    !
    !
    devices device fw01
    config
    snmp-server community NEW_READ
    !
    !
    devices device fw02
    config
    snmp-server community NEW_READ
    !
    !
    devices device internet-rtr
    config
    no snmp-server community INITIAL_READ RO
    no snmp-server community INITIAL_WRITE RW
    snmp-server community NEW_READ RO
    snmp-server community NEW_WRITE RW
    !
    !
    devices device leaf01-1
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device leaf01-2
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device leaf02-1
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device leaf02-2
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device spine01-1
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device spine01-2
    config
    no snmp-server community INITIAL_READ group network-operator
    no snmp-server community INITIAL_WRITE group network-admin
    snmp-server community NEW_READ group network-operator
    snmp-server community NEW_WRITE group network-admin
    !
    !
    devices device vm-sw01
    config
    no snmp-server community INITIAL_READ RO
    no snmp-server community INITIAL_WRITE RW
    snmp-server community NEW_READ RO
    snmp-server community NEW_WRITE RW
    !
    !
    devices device vm-sw02
    config
    no snmp-server community INITIAL_READ RO
    no snmp-server community INITIAL_WRITE RW
    snmp-server community NEW_READ RO
    snmp-server community NEW_WRITE RW
    ```
    </details>

## All done!
And that's it... you can now update community strings with ease!     
