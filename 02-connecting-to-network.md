# Connecting to the Network for the First Time 
You've now got NSO installed and you are ready to dive in and use it on your network, but what network?  

If you have a lab ready to go, feel free to use that - but we've included a [VIRL topology file](topology.virl) with this lab that we will use for our examples.  Feel free to fire, up your own instance of this VIRL network either using your own VIRL server, or go reserve one from [DevNet Sandbox - Multi-IOS Test Network](https://devnetsandbox.cisco.com/RM/Diagram/Index/6b023525-4e7f-4755-81ae-05ac500d464a?diagramType=Topology)

<a href="example-network.jpg" target="_blank">![](example-network-sm.jpeg)</a>

> The exmaple network provides a basic data center network of Nexus 9000 switches with multiple VRFs and VLANs, connected to a DMZ through an ASA firewall, where IOS XE routers handle a bit or routing, before an Edge ASAv firewall provides access to a simulated "Internet".  Also included are several different servers that can be used for testing network functionality.  

<details><summary>Devices Details</summary>

```bash
╒═══════════════════╤═════════════╤═════════╤═════════════╤════════════╤══════════════════════╤════════════════════╕
│ Node              │ Type        │ State   │ Reachable   │ Protocol   │ Management Address   │ External Address   │
╞═══════════════════╪═════════════╪═════════╪═════════════╪════════════╪══════════════════════╪════════════════════╡
│ admin-mgmt01      │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.167        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ admin-systems01   │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.151        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dev-app01         │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.154        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dev-data01        │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.153        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dev-web01         │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.161        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dmz-fw01-1        │ ASAv        │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.133        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dmz-rtr02-1       │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.135        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dmz-rtr02-2       │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.136        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dmz-sw01-1        │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.107        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dmz-sw01-2        │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.108        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dmz-sw02-1        │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.109        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ dmz-sw02-2        │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.110        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ fw-inside-sw01    │ IOSvL2      │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.122        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ fw-outside-sw01   │ IOSvL2      │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.123        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ fw01              │ ASAv        │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.145        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ fw02              │ ASAv        │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.146        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ guest-01          │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.164        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ internet-rtr      │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.171        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ internet-server01 │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.175        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ internet-server02 │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.176        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ leaf01-1          │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.103        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ leaf01-2          │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.104        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ leaf02-1          │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.105        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ leaf02-2          │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.106        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ prod-app01        │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.155        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ prod-data01       │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.162        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ prod-web01        │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.152        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ spine01-1         │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.101        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ spine01-2         │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.102        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ vm-sw01           │ IOSvL2      │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.121        │ N/A                │
├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
│ vm-sw02           │ IOSvL2      │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.115        │ N/A                │
╘═══════════════════╧═════════════╧═════════╧═════════════╧════════════╧══════════════════════╧════════════════════╛
```

</details>

## Starting the sample VIRL Network
Follow these steps to start the sample network. 

1. Reserve an instance of [DevNet Sandbox - Multi-IOS Test Network](https://devnetsandbox.cisco.com/RM/Diagram/Index/6b023525-4e7f-4755-81ae-05ac500d464a?diagramType=Topology). Pick **None** as the `Virl Simulation` before reserving.  This will start the lab with NO topology loaded in VIRL, we'll be starting our own. 

    ![](sbx-reserve-1.jpg)

    > It will take about 10 minutes for the lab to fully spin up.  

1. Once your lab is ready, use the VPN credentials you recieve via email, or find in the **OUTPUT** box in the Sandbox portal to connect to your lab. 
1. If you haven't already, clone down this repository to your local workstation. 

    ```bash
    git clone https://github.com/hpreston/nso-getting-started.git
    ```

1. Change into the code repository directory. 

    ```bash
    cd nso-getting-started
    ```

1. We will use the Python package `virlutils` to start the lab.  If you don't have it installed already, go ahead and do so now. 

    > Note: This guide assumes a basic knowledge of Python and doesn't explicitly state to use a virtual environment, or a version of Python to use.  Any recent version of Python and common working environment should work. 

    ```bash
    pip install virlutils
    ```

1. Verify that your lab and virlutils is working.

    ```bash
    # Check what version of VIRL is loaded (verify you can talk to server)
    virl version

    # Check if any simulations are running
    virl ls --all
    ```

    <details><summary>Output</summary>

    ```
    $virl version
    virlutils Version: 0.8.8
    VIRL Core Version: 0.10.37.32
    
    $ virl ls --all
    Running Simulations
    ╒══════════════╤══════════╤════════════════════════════╤═══════════╕
    │ Simulation   │ Status   │ Launched                   │ Expires   │
    ╞══════════════╪══════════╪════════════════════════════╪═══════════╡
    │ ~jumphost    │ ACTIVE   │ 2019-12-15T16:27:42.567768 │           │
    ╘══════════════╧══════════╧════════════════════════════╧═══════════╛
    ```

    </details>


    * If any simulation other than `~jumphost` is running, stop it with the following command. 

        ```bash
        virl down --sim-name {SIMULATION_NAME}
        ```

1. Start a simulation for this lab. 

    ```bash
    virl up
    ```

    <details><summary>Output</summary>

    ```
    Creating default environment from topology.virl
    ```

    </details>

1. It will take several minutes for the simulation to fully start, you can monitor it with the following command - waiting for all devices to show `REACHABLE`

    ```bash
    virl nodes
    ```

    <details><summary>Devices Details</summary>

    ```bash
    ╒═══════════════════╤═════════════╤═════════╤═════════════╤════════════╤══════════════════════╤════════════════════╕
    │ Node              │ Type        │ State   │ Reachable   │ Protocol   │ Management Address   │ External Address   │
    ╞═══════════════════╪═════════════╪═════════╪═════════════╪════════════╪══════════════════════╪════════════════════╡
    │ admin-mgmt01      │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.167        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ admin-systems01   │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.151        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dev-app01         │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.154        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dev-data01        │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.153        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dev-web01         │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.161        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dmz-fw01-1        │ ASAv        │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.133        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dmz-rtr02-1       │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.135        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dmz-rtr02-2       │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.136        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dmz-sw01-1        │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.107        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dmz-sw01-2        │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.108        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dmz-sw02-1        │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.109        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ dmz-sw02-2        │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.110        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ fw-inside-sw01    │ IOSvL2      │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.122        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ fw-outside-sw01   │ IOSvL2      │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.123        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ fw01              │ ASAv        │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.145        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ fw02              │ ASAv        │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.146        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ guest-01          │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.164        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ internet-rtr      │ CSR1000v    │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.171        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ internet-server01 │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.175        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ internet-server02 │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.176        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ leaf01-1          │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.103        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ leaf01-2          │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.104        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ leaf02-1          │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.105        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ leaf02-2          │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.106        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ prod-app01        │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.155        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ prod-data01       │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.162        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ prod-web01        │ server      │ ACTIVE  │ REACHABLE   │ ssh        │ 172.16.30.152        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ spine01-1         │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.101        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ spine01-2         │ NX-OSv 9000 │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.102        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ vm-sw01           │ IOSvL2      │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.121        │ N/A                │
    ├───────────────────┼─────────────┼─────────┼─────────────┼────────────┼──────────────────────┼────────────────────┤
    │ vm-sw02           │ IOSvL2      │ ACTIVE  │ REACHABLE   │ telnet     │ 172.16.30.115        │ N/A                │
    ╘═══════════════════╧═════════════╧═════════╧═════════════╧════════════╧══════════════════════╧════════════════════╛
    ```

    </details>


## Setting up and Starting NSO
As you learned during the installation step, a Local Install unpacks and prepares your system to run NSO, but doesn't actually start it up.  A Local Install allows the engineer to create an NSO "instance" tied to a project.  If you've done much work with Python, you can think of this like creating a Python virtual environment after installing Python.  Within this NSO instance, you will have different inventory, configuration, and code.  

1. First up, navigate to a directory where you'll want to setup this new NSO instance.  The lab's code repository includes a folder called `nso-instance` - this is a great place

    ```bash
    cd ~/code/nso-getting-started/nso-instance/
    ```

    > This folder has a `.gitignore` and `Makefile` contained within.  The ignore file is there to prevent committing NSO files to the repository, and the `Makefile` provides some helpful shortcuts and targets that you'll see shortly.  The folder `lab-configs/` contains a few NSO configuration files for reference or use in the lab. 

1. If you haven't already, be sure to "source" the `ncsrc` file.  
1. One of the included scripts with an NSO installation is `ncs-setup` which makes it very easy to create instances of NSO from a Local Install.  You can look at the `--help` for full details, but the two options we need to know are 
    * `--dest` - the directory where you want to setup NSO
    * `--package` - the NEDs you want to this NSO instance to have installed.  You can specify this option multiple times.  
1. Go ahead and run this command to setup an NSO instance in the current directory with the IOS, NX-OS, and ASA NEDs.  

    > You will want to use the name of the NED folder in `${NCS_DIR}/packages/neds` for the latest NED version you've got installed for the target platform.

    ```bash
    ncs-setup --package cisco-ios-cli-6.40 \
    --package cisco-nx-cli-5.13 \
    --package cisco-asa-cli-6.7 \
    --dest .
    ```

1. If you checkout the directory now you'll find several new files and folders have been created.  We won't go through them all in detail now, but a couple are handy to know about. 
    * `ncs.conf` - the NSO configuration file.  Used to customize aspects of the NSO instance.  The defaults are often perfect for projects like this. 
    * `packages/` - this directory will have symlinks to the NEDs that we referenced in the `--package` arguments at setup.  
    * `logs/` - this directory contains all the logs from NSO.  When troubleshooting, this is a handy directory. 

1. Last step is to "start" NSO.  Simply type the command `ncs`.  It will take a few seconds to run - and you won't get any explicit output unless there is a problem. 

    ```bash
    ncs
    ```


1. Jump into your NSO instance using the `ncs_cli` command.  This command takes several options - we'll use `-C` to specify we'd like a "Cisco style" command line interface (the default is a Juniper style), and `-u admin` to login as the "admin" user. 

    ```bash
    ncs_cli -C -u admin
    ```

    <details><summary>Output</summary>

    ```
    admin connected from 127.0.0.1 using console on HAPRESTO-M-Q0Y6
    admin@ncs#
    ```

    </details>

## Setting up Device Authentication
With NSO running, we now need to add our devices into it's "inventory" so that it can read and write data.  NSO uses "authgroups" to setup credentials for device access.  We'll setup a simple `authgroup` for our network.  

1. If not already, start the NSO CLI with `ncs_cli -C -u admin`. 
1. Enter into "config mode". 

    ```text
    config 
    ```

    <details><summary>OUTPUT</summary>

    ```
    Entering configuration mode terminal
    admin@ncs(config)#
    ```

    </details>

1. We will setup a new authgroup called "labadmin".  This group will use a default username/password combination of `cisco / cisco` for devices, with a "secondary" (ie 'enable') password also of `cisco`. These commands will set this up. 

    ```text 
    devices authgroups group labadmin
    default-map remote-name cisco
    default-map remote-password cisco
    default-map remote-secondary-password cisco
    ```

1. Entering this configuration into NSO "stages" it, but it isn't committed to the system until you use the command `commit`.  Before we do so, jump back ot the `top` of config mode, and look at the currently staged `config` with `show config`. 

    ```text 
    top 
    show config
    ```

    <details><summary>OUTPUT</summary>

    ```
    devices authgroups group labadmin
    default-map remote-name   cisco
    default-map remote-password $9$gGBuW7qSU5faSS/gkuyB1PXmh0Q2Da+gWPazNMq1SW8=
    default-map remote-secondary-password $9$gJEos3QbeiJsJe0xnZ3EoDv7r3eX3fXBhc0szTM2D64=
    ```

    </details>

    > This is a very handy way to see what configuration is ready to be committed before you finalize it.  Also, note that NSO uses a local encryption model to store the credentials in it's database.  

1. Now `commit` this to lock in the changes.  

> You can create as many `authgroups` as you need to for the different variations of authentication used across your network.  authgroups also can leverage SNMP or certificates in addition to traditional username/passwords .  Furthermore, NSO supports very powerful methods of RBAC for identifying and leveraging different authentication and authorization levels depending on the user logged into NSO.  These features are beyond scope for this introduction. 

## Adding Devices to Cisco NSO 
With our authgroup setup, let's go ahead and add our devices to the inventory.  To add a device you'll need the following information: 

* The device's address - IP or FQDN. 
* The protocol (ssh/telnet) and port (if non-standard) to connect to the device. 
* The device's authgroup to use (see above).
* The NED you'll use to connect to the device. 

> We'll walk through the first device step by step, then bulk add the rest of them. 

1. Start out back in `config` mode in NSO. 
1. We will first add `dmz-rtr02-1` using this information
    * Address: `172.16.30.135` 
    * Protocol: SSH
        * We will disable SSH host-key-verification (in production systems you can configure the host-key or just "fetch" to learn)
    * NED: `cisco-ios-cli-6.40`
1. Enter this configuration into NSO. 

    ```text
    devices device dmz-rtr01-1
    address 172.16.30.135
    authgroup labadmin
    device-type cli ned-id cisco-ios-cli-6.40
    device-type cli protocol ssh
    ssh host-key-verification none
    ```

1. Now `commit` this to NSO to lock it in. 
1. You should still be in the CLI context for the device, evident by the `prompt` or checking with `pwd`

    ```text 
    admin@ncs(config-device-dmz-rtr01-1)# pwd
    Current submode path:
    devices device dmz-rtr01-1
    ```

1. Let's see if we can connect to the device with NSO. 

    ```text 
    connect 
    ```

    <details><summary>OUTPUT</summary>

    ```
    result false
    info Device dmz-rtr01-1 is southbound locked
    ```

    </details>

1. Uh oh... we are currently "locked".  NSO's default mode for devices is a "locked state".  This prevents NSO from manipulating a device before the network admin is ready.  This is also a handy ability to "disable" a device that is currently in maintenance to prevent NSO from acting on it. "Unlock" it by changing it's `admin-state` and `commit` the change. 

    ```
    state admin-state unlocked
    commit
    ```

1. Now try to `connect` again. 

    ```text 
    connect
    ```

    <details><summary>OUTPUT</summary>

    ```
    result true
    info (admin) Connected to dmz-rtr01-1 - 172.16.30.135:22
    ```

    </details>

1. Excellent!  Now you just need to repeat the process for the remaining devices, and there are a few options.  
    1. You can manually type the commands for each device - great practice but a bit boring. 
    1. You can open up the file [`nso-inventory.cfg`](nso-instance/lab-configs/nso-inventory.cfg) from the `nso-instance` directory that has the configuration needed all ready for you and copy/paste it.  A bit faster, but not the best option. 
    1. You can use the `load merge` command from NSO to read the file into NSO and stage it for committing.  

1. Let's assume you want to use this third option, though do take a look at the file to see what commands are being ran.  The only major difference between device types is the `ned-id` value being used.  

    ```text 
    top
    load merge lab-configs/nso-inventory.cfg
    ```

    > Don't forget to use `show configuration` after `load merge` to verify what has been staged. 

1. Now just `commit` to complete adding these devices to NSO. 

    ```text
    commit
    ```

1. Now use `end` to exit from `config` mode.  
1. Verify the devices have been added to NSO.  

    ```text
    show devices list 
    ```

    <details><summary>OUTPUT</summary>

    ```
    NAME             ADDRESS        DESCRIPTION  NED ID             ADMIN STATE  
    ---------------------------------------------------------------------------
    dmz-fw01-1       172.16.30.133  -            cisco-asa-cli-6.7  unlocked     
    dmz-rtr01-1      172.16.30.135  -            cisco-ios-cli-6.40  unlocked     
    dmz-rtr02-1      172.16.30.135  -            cisco-ios-cli-6.40  unlocked     
    dmz-rtr02-2      172.16.30.136  -            cisco-ios-cli-6.40  unlocked     
    dmz-sw01-1       172.16.30.107  -            cisco-nx-cli-5.13   unlocked     
    dmz-sw01-2       172.16.30.108  -            cisco-nx-cli-5.13   unlocked     
    dmz-sw02-1       172.16.30.109  -            cisco-nx-cli-5.13   unlocked     
    dmz-sw02-2       172.16.30.110  -            cisco-nx-cli-5.13   unlocked     
    fw-inside-sw01   172.16.30.122  -            cisco-ios-cli-6.40  unlocked     
    fw-outside-sw01  172.16.30.123  -            cisco-ios-cli-6.40  unlocked     
    fw01             172.16.30.145  -            cisco-asa-cli-6.7  unlocked     
    fw02             172.16.30.146  -            cisco-asa-cli-6.7  unlocked     
    internet-rtr     172.16.30.171  -            cisco-ios-cli-6.40  unlocked     
    leaf01-1         172.16.30.103  -            cisco-nx-cli-5.13   unlocked     
    leaf01-2         172.16.30.104  -            cisco-nx-cli-5.13   unlocked     
    leaf02-1         172.16.30.105  -            cisco-nx-cli-5.13   unlocked     
    leaf02-2         172.16.30.106  -            cisco-nx-cli-5.13   unlocked     
    spine01-1        172.16.30.101  -            cisco-nx-cli-5.13   unlocked     
    spine01-2        172.16.30.102  -            cisco-nx-cli-5.13   unlocked     
    vm-sw01          172.16.30.121  -            cisco-ios-cli-6.40  unlocked     
    vm-sw02          172.16.30.115  -            cisco-ios-cli-6.40  unlocked 
    ```

    </details>

1. Finally, make sure NSO can communicate with these devices by attempting to `connect` to them all.  You can do this to all devices very easily like this.  

    ```text
    devices connect
    ```

    <details><summary>OUTPUT</summary>

    ```
    connect-result {
        device dmz-fw01-1
        result true
        info (admin) Connected to dmz-fw01-1 - 172.16.30.133:22
    }
    connect-result {
        device dmz-rtr01-1
        result true
        info (admin) Connected to dmz-rtr01-1 - 172.16.30.135:22
    }
    connect-result {
        device dmz-rtr02-1
        result true
        info (admin) Connected to dmz-rtr02-1 - 172.16.30.135:22
    }
    connect-result {
        device dmz-rtr02-2
        result true
        info (admin) Connected to dmz-rtr02-2 - 172.16.30.136:22
    }
    connect-result {
        device dmz-sw01-1
        result true
        info (admin) Connected to dmz-sw01-1 - 172.16.30.107:22
    }
    connect-result {
        device dmz-sw01-2
        result true
        info (admin) Connected to dmz-sw01-2 - 172.16.30.108:22
    }
    connect-result {
        device dmz-sw02-1
        result true
        info (admin) Connected to dmz-sw02-1 - 172.16.30.109:22
    }
    connect-result {
        device dmz-sw02-2
        result true
        info (admin) Connected to dmz-sw02-2 - 172.16.30.110:22
    }
    connect-result {
        device fw-inside-sw01
        result true
        info (admin) Connected to fw-inside-sw01 - 172.16.30.122:22
    }
    connect-result {
        device fw-outside-sw01
        result true
        info (admin) Connected to fw-outside-sw01 - 172.16.30.123:22
    }
    connect-result {
        device fw01
        result true
        info (admin) Connected to fw01 - 172.16.30.145:22
    }
    connect-result {
        device fw02
        result true
        info (admin) Connected to fw02 - 172.16.30.146:22
    }
    connect-result {
        device internet-rtr
        result true
        info (admin) Connected to internet-rtr - 172.16.30.171:22
    }
    connect-result {
        device leaf01-1
        result true
        info (admin) Connected to leaf01-1 - 172.16.30.103:22
    }
    connect-result {
        device leaf01-2
        result true
        info (admin) Connected to leaf01-2 - 172.16.30.104:22
    }
    connect-result {
        device leaf02-1
        result true
        info (admin) Connected to leaf02-1 - 172.16.30.105:22
    }
    connect-result {
        device leaf02-2
        result true
        info (admin) Connected to leaf02-2 - 172.16.30.106:22
    }
    connect-result {
        device spine01-1
        result true
        info (admin) Connected to spine01-1 - 172.16.30.101:22
    }
    connect-result {
        device spine01-2
        result true
        info (admin) Connected to spine01-2 - 172.16.30.102:22
    }
    connect-result {
        device vm-sw01
        result true
        info (admin) Connected to vm-sw01 - 172.16.30.121:22
    }
    connect-result {
        device vm-sw02
        result true
        info (admin) Connected to vm-sw02 - 172.16.30.115:22
    }
    ```

    </details>

## "Learning" Current Network State (sync-from)
Now NSO can successfully connect to the network, but it hasn't "learned" the current network configuration deployed to the devices.  Let's fix that.  

1. First, see for yourself that there is no real device level configuration within NSO yet by looking at a device. 

    ```text 
    show running-config devices device leaf01-1 config
    ```

    <details><summary>OUTPUT</summary>

    ```
    config
    no feature ssh
    no feature telnet
    ```

    </details>

    > Checkout a few other devices.  Differnet NEDs have different "default states". 

1. NSO makes it easy to `sync-from` the network devices to "learn" the current configuration.  Go ahead and do that to all devices.  

    > **NOTE: Be sure you do *NOT* enter `devices sync-to` by mistake.  This will have NSO attempt to overwrite the configuration on the devices with the "blank" configuration that NSO currently has for the devices.  This will be bad.**

    ```text
    devices sync-from 
    ```

    <details><summary>OUTPUT</summary>

    ```
    sync-result {
        device dmz-fw01-1
        result true
    }
    sync-result {
        device dmz-rtr01-1
        result true
    }
    sync-result {
        device dmz-rtr02-1
        result true
    }
    sync-result {
        device dmz-rtr02-2
        result true
    }
    sync-result {
        device dmz-sw01-1
        result true
    }
    sync-result {
        device dmz-sw01-2
        result true
    }
    sync-result {
        device dmz-sw02-1
        result true
    }
    sync-result {
        device dmz-sw02-2
        result true
    }
    sync-result {
        device fw-inside-sw01
        result true
    }
    sync-result {
        device fw-outside-sw01
        result true
    }
    sync-result {
        device fw01
        result true
    }
    sync-result {
        device fw02
        result true
    }
    sync-result {
        device internet-rtr
        result true
    }
    sync-result {
        device leaf01-1
        result true
    }
    sync-result {
        device leaf01-2
        result true
    }
    sync-result {
        device leaf02-1
        result true
    }
    sync-result {
        device leaf02-2
        result true
    }
    sync-result {
        device spine01-1
        result true
    }
    sync-result {
        device spine01-2
        result true
    }
    sync-result {
        device vm-sw01
        result true
    }
    sync-result {
        device vm-sw02
        result true
    }
    ```

    </details>

1. Now re-check the NSO `running-configuration` of the devices.  

    ```text
    show running-config devices device leaf01-1 config   
    ```

    <details><summary>OUTPUT</summary>

    ```
    show running-config devices device leaf01-1 config   
    devices device leaf01-1
    config
    ! Command: show running-config Running configuration last done at: Sun Dec 15 17:21:37 2019 Time: Mon Dec 16 02:03:17 2019
    hostname leaf01-1
    cfs eth distribute
    feature hsrp
    feature ospf
    feature vpc
    feature interface-vlan
    feature lacp
    no feature ssh
    feature telnet
    username admin password 5 $1$KuOSBsvW$Cy0TSD..gEBGBPjzpDgf51 role network-admin
    username cisco password 5 $1$Nk7ZkwH0$fyiRmMMfIheqE3BqvcL0C1 role network-admin
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
    vrf context backdoor
    !
    vrf context dmz
    ip route 0.0.0.0/0 10.225.250.4
    !
    vrf context guest
    !
    vrf context management
    ip route 0.0.0.0/0 172.16.30.254
    !
    vrf context trusted
    ip route 0.0.0.0/0 10.225.250.4
    !
    vpc domain 10
    peer-gateway
    peer-switch
    !
    vdc leaf01-1 id 1
    limit-resource vlan minimum 16 maximum 4094
    limit-resource port-channel minimum 0 maximum 511
    limit-resource u4route-mem minimum 128 maximum 128
    limit-resource u6route-mem minimum 96 maximum 96
    limit-resource m4route-mem minimum 58 maximum 58
    limit-resource m6route-mem minimum 8 maximum 8
    !
    interface Ethernet1/1
    description VPC Peer Link
    channel-group 1 mode active
    switchport mode trunk
    !
    interface Ethernet1/10
    !
    interface Ethernet1/100
    !
    interface Ethernet1/101
    !
    interface Ethernet1/102
    !
    interface Ethernet1/103
    !
    interface Ethernet1/104
    !
    interface Ethernet1/105
    !
    interface Ethernet1/106
    !
    interface Ethernet1/107
    !
    interface Ethernet1/108
    !
    interface Ethernet1/109
    !
    interface Ethernet1/11
    description Interswitch Fabric Link
    channel-group 3 mode active
    switchport mode trunk
    !
    interface Ethernet1/110
    !
    interface Ethernet1/111
    !
    interface Ethernet1/112
    !
    interface Ethernet1/113
    !
    interface Ethernet1/114
    !
    interface Ethernet1/115
    !
    interface Ethernet1/116
    !
    interface Ethernet1/117
    !
    interface Ethernet1/118
    !
    interface Ethernet1/119
    !
    interface Ethernet1/12
    !
    interface Ethernet1/120
    !
    interface Ethernet1/121
    !
    interface Ethernet1/122
    !
    interface Ethernet1/123
    !
    interface Ethernet1/124
    !
    interface Ethernet1/125
    !
    interface Ethernet1/126
    !
    interface Ethernet1/127
    !
    interface Ethernet1/128
    !
    interface Ethernet1/13
    !
    interface Ethernet1/14
    !
    interface Ethernet1/15
    !
    interface Ethernet1/16
    !
    interface Ethernet1/17
    !
    interface Ethernet1/18
    !
    interface Ethernet1/19
    !
    interface Ethernet1/2
    description VPC Peer Link
    channel-group 1 mode active
    switchport mode trunk
    !
    interface Ethernet1/20
    !
    interface Ethernet1/21
    !
    interface Ethernet1/22
    !
    interface Ethernet1/23
    !
    interface Ethernet1/24
    !
    interface Ethernet1/25
    !
    interface Ethernet1/26
    !
    interface Ethernet1/27
    !
    interface Ethernet1/28
    !
    interface Ethernet1/29
    !
    interface Ethernet1/3
    description Interswitch Fabric Link
    channel-group 2 mode active
    switchport mode trunk
    !
    interface Ethernet1/30
    !
    interface Ethernet1/31
    !
    interface Ethernet1/32
    !
    interface Ethernet1/33
    !
    interface Ethernet1/34
    !
    interface Ethernet1/35
    !
    interface Ethernet1/36
    !
    interface Ethernet1/37
    !
    interface Ethernet1/38
    !
    interface Ethernet1/39
    !
    interface Ethernet1/4
    description Interswitch Fabric Link
    channel-group 2 mode active
    switchport mode trunk
    !
    interface Ethernet1/40
    !
    interface Ethernet1/41
    !
    interface Ethernet1/42
    !
    interface Ethernet1/43
    !
    interface Ethernet1/44
    !
    interface Ethernet1/45
    !
    interface Ethernet1/46
    !
    interface Ethernet1/47
    !
    interface Ethernet1/48
    !
    interface Ethernet1/49
    !
    interface Ethernet1/5
    !
    interface Ethernet1/50
    !
    interface Ethernet1/51
    !
    interface Ethernet1/52
    !
    interface Ethernet1/53
    !
    interface Ethernet1/54
    !
    interface Ethernet1/55
    !
    interface Ethernet1/56
    !
    interface Ethernet1/57
    !
    interface Ethernet1/58
    !
    interface Ethernet1/59
    !
    interface Ethernet1/6
    !
    interface Ethernet1/60
    !
    interface Ethernet1/61
    !
    interface Ethernet1/62
    !
    interface Ethernet1/63
    !
    interface Ethernet1/64
    !
    interface Ethernet1/65
    !
    interface Ethernet1/66
    !
    interface Ethernet1/67
    !
    interface Ethernet1/68
    !
    interface Ethernet1/69
    !
    interface Ethernet1/7
    !
    interface Ethernet1/70
    !
    interface Ethernet1/71
    !
    interface Ethernet1/72
    !
    interface Ethernet1/73
    !
    interface Ethernet1/74
    !
    interface Ethernet1/75
    !
    interface Ethernet1/76
    !
    interface Ethernet1/77
    !
    interface Ethernet1/78
    !
    interface Ethernet1/79
    !
    interface Ethernet1/8
    !
    interface Ethernet1/80
    !
    interface Ethernet1/81
    !
    interface Ethernet1/82
    !
    interface Ethernet1/83
    !
    interface Ethernet1/84
    !
    interface Ethernet1/85
    !
    interface Ethernet1/86
    !
    interface Ethernet1/87
    !
    interface Ethernet1/88
    !
    interface Ethernet1/89
    !
    interface Ethernet1/9
    !
    interface Ethernet1/90
    !
    interface Ethernet1/91
    !
    interface Ethernet1/92
    !
    interface Ethernet1/93
    !
    interface Ethernet1/94
    !
    interface Ethernet1/95
    !
    interface Ethernet1/96
    !
    interface Ethernet1/97
    !
    interface Ethernet1/98
    !
    interface Ethernet1/99
    !
    interface mgmt0
    description OOB Management
    ip address 172.16.30.103/24
    ip redirects
    vrf member management
    !
    interface port-channel1
    spanning-tree port type network
    switchport mode trunk
    vpc peer-link
    !
    interface port-channel2
    description Interswitch Fabric Link
    switchport mode trunk
    vpc 2
    !
    interface port-channel3
    description Interswitch Fabric Link
    switchport mode trunk
    vpc 3
    !
    interface Vlan1
    no ip redirects
    !
    interface Vlan10
    ip address 10.225.250.2/29
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member trusted
    hsrp version 2
    hsrp 1
        ip       10.225.250.1
        preempt
        priority 110
    !
    !
    interface Vlan11
    ip address 10.225.1.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member trusted
    hsrp version 2
    hsrp 1
        ip       10.225.1.1
        preempt
        priority 110
    !
    !
    interface Vlan12
    ip address 10.225.2.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member trusted
    hsrp version 2
    hsrp 1
        ip       10.225.2.1
        preempt
        priority 110
    !
    !
    interface Vlan100
    ip address 10.225.250.34/29
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member dmz
    hsrp version 2
    hsrp 1
        ip       10.225.250.33
        preempt
        priority 110
    !
    !
    interface Vlan101
    ip address 10.225.9.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member dmz
    hsrp version 2
    hsrp 1
        ip       10.225.9.1
        preempt
        priority 110
    !
    !
    interface Vlan102
    ip address 10.225.10.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member trusted
    hsrp version 2
    hsrp 1
        ip       10.225.10.1
        preempt
        priority 110
    !
    !
    interface Vlan103
    ip address 10.225.11.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member trusted
    hsrp version 2
    hsrp 1
        ip       10.225.11.1
        preempt
        priority 110
    !
    !
    interface Vlan201
    ip address 10.225.17.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member dmz
    hsrp version 2
    hsrp 1
        ip       10.225.17.1
        preempt
        priority 110
    !
    !
    interface Vlan202
    ip address 10.225.18.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member trusted
    hsrp version 2
    hsrp 1
        ip       10.225.18.1
        preempt
        priority 110
    !
    !
    interface Vlan203
    ip address 10.225.19.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member trusted
    hsrp version 2
    hsrp 1
        ip       10.225.19.1
        preempt
        priority 110
    !
    !
    interface Vlan900
    ip address 10.225.250.50/29
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member guest
    hsrp version 2
    hsrp 1
        ip       10.225.250.49
        preempt
        priority 110
    !
    !
    interface Vlan901
    ip address 10.225.49.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member guest
    hsrp version 2
    hsrp 1
        ip       10.225.49.1
        preempt
        priority 110
    !
    !
    interface Vlan3001
    ip address 172.16.30.2/24
    no ip redirects
    ip router ospf 1 area 0.0.0.0
    no shutdown
    vrf member backdoor
    hsrp version 2
    hsrp 1
        ip       172.16.30.1
        preempt
        priority 110
    !
    !
    router ospf 1
    vrf backdoor
        passive-interface default
    !
    vrf dmz
        passive-interface default
    !
    vrf guest
        passive-interface default
    !
    vrf trusted
        passive-interface default
    ```

    </details>

    > Explore the configs of other devices. 

## Grouping Devices Together
This final step in connecting to the network isn't required, but will make working with the network simpler.  What we will do is create `device-groups` to organize our devices into logical groups that we may want to apply similar configuration, or take similar action upon.  

Let's create the following groups 

* Network Operating System Based Groups
    * `IOS-DEVICES`
    * `NXOS-DEVICES`
    * `ASA-DEVICES`
* Point in Network Based Groups
    * `INTERNAL-DEVICES`
    * `DMZ-DEVICES`
    * `OUTSIDE-DEVICES`

> You have full flexibility to create as many groups as you want.  And groups can contain other groups for nesting.  

1. Enter config mode.
1. Groups are very easy to create, here's the `IOS-DEVICES` group. 

    ```text
    devices device-group IOS-DEVICES
    device-name dmz-rtr02-1
    device-name dmz-rtr02-2
    device-name internet-rtr
    device-name fw-inside-sw01
    device-name fw-outside-sw01
    device-name vm-sw01
    device-name vm-sw02
    ```

1. And the `NXOS-DEVICES` group. 

    ```text
    devices device-group NXOS-DEVICES
    device-name dmz-sw01-1
    device-name dmz-sw01-2
    device-name dmz-sw02-1
    device-name dmz-sw02-2
    device-name leaf01-1
    device-name leaf01-2
    device-name leaf02-1
    device-name leaf02-2
    device-name spine01-1
    device-name spine01-2
    ```

1. And the `ASA-DEVICES` group. 

    ```text
    devices device-group ASA-DEVICES
    device-name dmz-fw01-1
    device-name fw01
    device-name fw02
    ```

1. And here are all the Point in Network Groups 

    ```text
    devices device-group INTERNAL-DEVICES
    device-name fw-inside-sw01
    device-name vm-sw01
    device-name vm-sw02
    device-name leaf01-1
    device-name leaf01-2
    device-name leaf02-1
    device-name leaf02-2
    device-name spine01-1
    device-name spine01-2

    devices device-group DMZ-DEVICES
    device-name dmz-rtr02-1
    device-name dmz-rtr02-2
    device-name fw-outside-sw01
    device-name dmz-sw01-1
    device-name dmz-sw01-2
    device-name dmz-sw02-1
    device-name dmz-sw02-2
    device-name dmz-fw01-1
    device-name fw01
    device-name fw02

    devices device-group OUTSIDE-DEVICES
    device-name internet-rtr
    ```

1. And one last group that will include `ALL` our devices.  

    ```
    devices device-group ALL
    device-group ASA-DEVICES
    device-group IOS-DEVICES
    device-group NXOS-DEVICES 
    ```

1. And now `commit` them to save the changes.  

    ```
    commit
    end
    ```

1. We'll use these groups more later in the lab, but here's an example of using them to quickly check the sync status of all internal devices.  

    ```
    devices device-group INTERNAL-DEVICES check-sync
    ```

    <details><summary>Output</summary>

    ```
    sync-result {
        device fw-inside-sw01
        result in-sync
    }
    sync-result {
        device leaf01-1
        result in-sync
    }
    sync-result {
        device leaf01-2
        result in-sync
    }
    sync-result {
        device leaf02-1
        result in-sync
    }
    sync-result {
        device leaf02-2
        result in-sync
    }
    sync-result {
        device spine01-1
        result in-sync
    }
    sync-result {
        device spine01-2
        result in-sync
    }
    sync-result {
        device vm-sw01
        result in-sync
    }
    sync-result {
        device vm-sw02
        result in-sync
    }
    ```

    </details>
