# Installing Cisco NSO 
As with any piece of software, before you can use it, you must install it.  Let's run through what that takes! 

## Where to get the code? 
Cisco makes NSO available for anyone to use and try in a non-production use completely free on [DevNet](https://developer.cisco.com).  Head on over to [https://developer.cisco.com/docs/nso/#!getting-nso](https://developer.cisco.com/docs/nso/#!getting-nso) and download the version you need.  

## Pre-requisites 
### Operating System 
Cisco NSO can run on macOS or on Linux systems.  If you are a Window's user (or if you don't wish to install it natively on your laptop), you can install NSO on a Linux virtual machine.  For small lab networks you'll only need a single vCPU and 2-4 GB of RAM. If you use Vagrant to manager local Linux virtual machines, there is a NSO Vagrant file as well: [https://github.com/NSO-developer/nso-vagrant](https://github.com/NSO-developer/nso-vagrant).

NSO also runs quite well within Docker Containers, and you can find resources for that at [gitlab.com/nso-developer/nso-docker](https://gitlab.com/nso-developer/nso-docker).

### Java 
NSO requires Java to be installed to function.  Any method of installing Java should work, but here are some simple methods.  

* macOS 
    * [Homebrew](https://formulae.brew.sh/cask/java#default) - `brew cask install java`
* Linux 
    * With `apt` - `apt install default-jre`
    * With `yum` - `yum install java-1.8.0-openjdk`

### Apache Ant 
[Apache Ant](https://ant.apache.org/) is a Java library and tool for building files and other dependencies.  You'll need to install it as well.  Here are some ways to do so. 

* macOS 
    * [Homebrew](https://formulae.brew.sh/formula/ant#default) - `brew install ant`
* Linux 
    * With `apt` - `apt install ante`
    * With `yum` - `yum install ant`


## Local vs System Installation 
Before you install NSO onto your system, you need to decide whether to do a "System" or "Local" installation.  Here's a simple breakdown of the two.  

* **System Install** is used when installing NSO for a centralized, "always-on", production grade purpose.  Is configured as a system daemon that would start and end with the underlying operating system.  
* **Local Install** is used for development, lab, and evaluation purposes. It unpacks all the application components, including docs and examples, and allows the user to instantiate and start instances of NSO on demand.  A Local Install on a single workstation can be used by the engineer to run multiple, unrelated instances of NSO for different labs and demos. 

As we are just getting started, we'll be using a "Local Installation" of NSO.  

## Performing a Local Installation
Once you've installed the pre-reqs and downloaded the NSO installation file for your operating system, you are ready to do your installation.  

1. Open up a terminal and navigate to the directory where you downloaded the installer. 

    ```bash
    cd ~/Downloads
    ll nso*.bin
    -rw-r--r--@ 1 hapresto  staff   199M Dec 15 11:45 nso-5.3.darwin.x86_64.installer.bin
    -rw-r--r--@ 1 hapresto  staff   199M Dec 15 11:45 nso-5.3.darwin.x86_64.signed.bin
    ```

    > If your file is a `signed.bin` file, this means that the file you downloaded has been digitally signed by Cisco, and when you execute it you'll verify the signature and unpack the `installer.bin`.  If you have the `installer.bin` skip down past the `signed` steps.

1. First step, you need to make the file you downloaded "executable". 
    * *Your file name may be different, just update appropriately.*

    ```bash
    chmod +x nso-5.3.darwin.x86_64.signed.bin

    ll nso-5.3.darwin.x86_64.signed.bin
    -rwxr-xr-x@ 1 hapresto  staff   199M Dec 15 11:45 nso-5.3.darwin.x86_64.signed.bin
    ```

1. Now go ahead and "run" the `signed.bin` to verify the certificate and extract the files.  

    ```bash
    ./nso-5.3.darwin.x86_64.signed.bin 

    # Output
    Unpacking...
    Verifying signature...
    Downloading CA certificate from http://www.cisco.com/security/pki/certs/crcam2.cer ...
    Successfully downloaded and verified crcam2.cer.
    Downloading SubCA certificate from http://www.cisco.com/security/pki/certs/innerspace.cer ...
    Successfully downloaded and verified innerspace.cer.
    Successfully verified root, subca and end-entity certificate chain.
    Successfully fetched a public key from tailf.cer.
    Successfully verified the signature of nso-5.3.darwin.x86_64.installer.bin using tailf.cer
    ```

1. If it all comes back green, you're in good shape and ready to install. First let's see what was unpacked.  

    ```bash
    ll

    # Output
    -rw-r--r--  1 hapresto  staff   1.8K Nov 29 06:05 README.signature
    -rw-r--r--  1 hapresto  staff    12K Nov 29 06:05 cisco_x509_verify_release.py
    -rwxr-xr-x  1 hapresto  staff   199M Nov 29 05:55 nso-5.3.darwin.x86_64.installer.bin
    -rw-r--r--  1 hapresto  staff   256B Nov 29 06:05 nso-5.3.darwin.x86_64.installer.bin.signature
    -rwxr-xr-x@ 1 hapresto  staff   199M Dec 15 11:45 nso-5.3.darwin.x86_64.signed.bin
    -rw-r--r--  1 hapresto  staff   1.4K Nov 29 06:05 tailf.cer
    ```

1. In addition to the `signed.bin` file you started with, you'll now have: 
    * `README.signature` - details on the verification 
    * `cisco_x509_verify_release.py` - Python script to verify the file 
    * `nso-5.3.darwin.x86_64.installer.bin` - the NSO installation file 
    * `nso-5.3.darwin.x86_64.installer.bin.signature` - the signature file for the installer 
    * `tailf.cer` - public cert for verification 

1. First, checkout the `--help` on the installer.  Notice the two options for `--local-install` or  `--system-install`

    ```bash
    ./nso-5.3.darwin.x86_64.installer.bin --help

    # Output 
    This is the NCS installation script.

    Usage: ./nso-5.3.darwin.x86_64.installer.bin [--local-install] LocalInstallDir

    Installs NCS in the LocalInstallDir directory only.
    This is convenient for test and development purposes.

    Usage: ./nso-5.3.darwin.x86_64.installer.bin --system-install [--install-dir InstallDir]
        [--config-dir ConfigDir] [--run-dir RunDir] [--log-dir LogDir]
        [--run-as-user User] [--keep-ncs-setup] [--non-interactive]

    Does a system install of NCS, suitable for deployment.
    Static files are installed in InstallDir/ncs-<vsn>.
    The first time --system-install is used, the ConfigDir,
    RunDir, and LogDir directories are also created and
    populated for config files, run-time state files, and log files,
    respectively, and an init script for start of NCS at system boot
    and user profile scripts are installed. Defaults are:

    InstallDir - /opt/ncs
    ConfigDir  - /etc/ncs
    RunDir     - /var/opt/ncs
    LogDir     - /var/log/ncs

    By default, the system install will run NCS as the root user.
    If the --run-as-user option is given, the system install will
    instead run NCS as the given user. The user will be created if
    it does not already exist.

    If the --non-interactive option is given, the installer will
    proceed with potentially disruptive changes (e.g. modifying or
    removing existing files) without asking for confirmation.
    ```

1. For the installation directory or `LocalInstallDir`, the recommendation is to install it into your $HOME directory into a folder called `~/nso-VERSION`.  So if our version is `5.3`, our directory will be `~/nso-5.3`. 
1. Run the installer with this directory. 

    ```bash
    ./nso-5.3.darwin.x86_64.installer.bin --local-install ~/nso-5.3

    # Output
    INFO  Using temporary directory /var/folders/90/n5sbctr922336_0jrzhb54400000gn/T//ncs_installer.93831 to stage NCS installation bundle
    INFO  Unpacked ncs-5.3 in /Users/hapresto/nso-5.3
    INFO  Found and unpacked corresponding DOCUMENTATION_PACKAGE
    INFO  Found and unpacked corresponding EXAMPLE_PACKAGE
    INFO  Found and unpacked corresponding JAVA_PACKAGE
    INFO  Generating default SSH hostkey (this may take some time)
    INFO  SSH hostkey generated
    INFO  Environment set-up generated in /Users/hapresto/nso-5.3/ncsrc
    INFO  NSO installation script finished
    INFO  Found and unpacked corresponding NETSIM_PACKAGE
    INFO  NCS installation complete
    ```

1. That's it, you're installed.

## Exploring the Installation
Before we startup NSO, let's just look at what we have.  

Go ahead and `cd` into the new installation directory.  

```bash
cd ~/nso-5.3
```

### Documentation 
Along with the binaries, NSO installs a full set of documentation available in the `doc/` folder.  

```bash
ll doc/
drwxr-xr-x   5 hapresto  staff   160B Nov 29 05:19 api/
drwxr-xr-x  14 hapresto  staff   448B Nov 29 05:19 html/
-rw-r--r--   1 hapresto  staff   202B Nov 29 05:19 index.html
drwxr-xr-x  17 hapresto  staff   544B Nov 29 05:19 pdf/
```

Feel free to open up the `index.html` file in your favorite browser and poke around. You'll find installation, admin, user, development, and more guides available for you to jump right into. 

### Examples 
An NSO Local Install also comes with **A LOT** of examples of a variety of different types of ways you can use NSO.  Many of these touch on advanced topics, but there are plenty of basic ones as well.  

```bash
ll examples.ncs/

# Output
-rw-r--r--   1 hapresto  staff   1.0K Nov 29 05:17 README
drwxr-xr-x   4 hapresto  staff   128B Nov 29 04:50 datacenter/
drwxr-xr-x   3 hapresto  staff    96B Nov 29 04:50 generic-ned/
drwxr-xr-x   4 hapresto  staff   128B Nov 29 04:50 getting-started/
drwxr-xr-x   7 hapresto  staff   224B Nov 29 04:50 service-provider/
drwxr-xr-x   3 hapresto  staff    96B Nov 29 04:50 snmp-ned/
drwxr-xr-x  11 hapresto  staff   352B Nov 29 05:17 snmp-notification-receiver/
drwxr-xr-x   5 hapresto  staff   160B Nov 29 04:50 web-server-farm/```
```

### NEDs or Network Element Drivers
In order to "talk to" the network, NSO uses NEDs.  Cisco has NEDs for hundreds of different devices available for customers, and several are included in the installer.  Let's see what they are.  

```bash
ll 

# Output
drwxr-xr-x  13 hapresto  staff   416B Nov 29 05:17 a10-acos-cli-3.0/
drwxr-xr-x  12 hapresto  staff   384B Nov 29 05:17 alu-sr-cli-3.4/
drwxr-xr-x  13 hapresto  staff   416B Nov 29 05:17 cisco-asa-cli-6.6/
drwxr-xr-x  12 hapresto  staff   384B Nov 29 05:17 cisco-ios-cli-3.0/
drwxr-xr-x  12 hapresto  staff   384B Nov 29 05:17 cisco-ios-cli-3.8/
drwxr-xr-x  13 hapresto  staff   416B Nov 29 05:17 cisco-iosxr-cli-3.0/
drwxr-xr-x  13 hapresto  staff   416B Nov 29 05:17 cisco-iosxr-cli-3.5/
drwxr-xr-x  13 hapresto  staff   416B Nov 29 05:17 cisco-nx-cli-3.0/
drwxr-xr-x  13 hapresto  staff   416B Nov 29 05:17 dell-ftos-cli-3.0/
drwxr-xr-x  10 hapresto  staff   320B Nov 29 05:17 juniper-junos-nc-3.0/
```

Here you can see there are NEDs for Cisco ASA, IOS, IOS XR, and NX-OS.  Also included are NEDs for other vendors including Juniper JunOS, A10, ALU and Dell. 

> Note: The NEDs included in the installer are intended for evaluation, demonstration, and used with the included `examples.ncs` that are also included.  These are not the latest versions available, and often don't have all the features available in production NEDs. 

#### Installing New NED Versions 
Cisco also makes additional versions of some NEDs available on DevNet for evaluation and non-production use.  You can find them with the NSO downloads [here](https://developer.cisco.com/docs/nso/#!getting-nso).  

For this getting started lab we'll be leveraging Cisco IOS, NX-OS and ASA devices.  Go ahead and download the updated NEDs from DevNet and we'll install them now!  

> NOTE: the specific file names and versions you download maybe different from this guide.  Update the paths appropriately.  

1. Like the NSO installer, the NEDs are signed bin files that need to be run to validate the download and extract the new code. 

1. First, let's see the downloaded files - update the path for where your downloads are 

    ```bash
    cd ~/Downloads/
    ls -l ncs*.bin

    # Output 
    -rw-r--r--@ 1 hapresto  staff   9708091 Dec 18 12:05 ncs-5.3-cisco-asa-6.7.7.signed.bin
    -rw-r--r--@ 1 hapresto  staff  51233042 Dec 18 12:06 ncs-5.3-cisco-ios-6.42.1.signed.bin
    -rw-r--r--@ 1 hapresto  staff   8400190 Dec 18 12:05 ncs-5.3-cisco-nx-5.13.1.1.signed.bin
    ```

1. Now make them "executable" 

    ```bash
    chmod +x ncs*.bin
    ls -l ncs*.bin
    ```

    <details><summary>Output - now executable</summary>

    ```bash
    -rwxr-xr-x@ 1 hapresto  staff   9708091 Dec 18 12:05 ncs-5.3-cisco-asa-6.7.7.signed.bin
    -rwxr-xr-x@ 1 hapresto  staff  51233042 Dec 18 12:06 ncs-5.3-cisco-ios-6.42.1.signed.bin
    -rwxr-xr-x@ 1 hapresto  staff   8400190 Dec 18 12:05 ncs-5.3-cisco-nx-5.13.1.1.signed.bin
    ```

    </details>

1. Run each file. 

    ```bash
    ./ncs-5.3-cisco-nx-5.13.1.1.signed.bin
    ```

    <details><summary>Output</summary>

    ```bash
    Unpacking...
    Verifying signature...
    Downloading CA certificate from http://www.cisco.com/security/pki/certs/crcam2.cer ...
    Successfully downloaded and verified crcam2.cer.
    Downloading SubCA certificate from http://www.cisco.com/security/pki/certs/innerspace.cer ...
    Successfully downloaded and verified innerspace.cer.
    Successfully verified root, subca and end-entity certificate chain.
    Successfully fetched a public key from tailf.cer.
    Successfully verified the signature of ncs-5.3-cisco-nx-5.13.1.1.tar.gz using tailf.cer
    ```

    </details>

    > Repeat for all three files 

1. You now have 3 tarballs (.tar.gz) files.  These are compressed versions of the NEDs. 

    ```bash 
    ls -l ncs*.tar.gz
    ```

    <details><summary>Output</summary>

    ```bash
    -rw-r--r--  1 hapresto  staff   9704896 Dec 12 21:11 ncs-5.3-cisco-asa-6.7.7.tar.gz
    -rw-r--r--  1 hapresto  staff  51260488 Dec 13 22:58 ncs-5.3-cisco-ios-6.42.1.tar.gz
    -rw-r--r--  1 hapresto  staff   8409288 Dec 18 09:09 ncs-5.3-cisco-nx-5.13.1.1.tar.gz
    ```

    </details>

1. Navigate to the `packages/neds` directory for your local-install. 

    ```bash
    cd ~/nso-5.3/packages/neds
    ```

1. All we need to do now is extract the tarballs into this directory. 

    > Update the path and file name for the NED versions you downloaded

    ```bash
    tar -zxvf ~/Downloads/ncs-5.3-cisco-nx-5.13.1.1.tar.gz
    tar -zxvf ~/Downloads/ncs-5.3-cisco-ios-6.42.1.tar.gz
    tar -zxvf ~/Downloads/ncs-5.3-cisco-asa-6.7.7.tar.gz

    ls -l
    ```

    <details><summary>Output</summary>

    ```bash
    drwxr-xr-x  13 hapresto  staff   416 Nov 29 05:17 a10-acos-cli-3.0
    drwxr-xr-x  12 hapresto  staff   384 Nov 29 05:17 alu-sr-cli-3.4
    drwxr-xr-x  13 hapresto  staff   416 Nov 29 05:17 cisco-asa-cli-6.6
    drwxr-xr-x  13 hapresto  staff   416 Dec 12 21:11 cisco-asa-cli-6.7
    drwxr-xr-x  12 hapresto  staff   384 Nov 29 05:17 cisco-ios-cli-3.0
    drwxr-xr-x  12 hapresto  staff   384 Nov 29 05:17 cisco-ios-cli-3.8
    drwxr-xr-x  13 hapresto  staff   416 Dec 13 22:58 cisco-ios-cli-6.42
    drwxr-xr-x  13 hapresto  staff   416 Nov 29 05:17 cisco-iosxr-cli-3.0
    drwxr-xr-x  13 hapresto  staff   416 Nov 29 05:17 cisco-iosxr-cli-3.5
    drwxr-xr-x  13 hapresto  staff   416 Nov 29 05:17 cisco-nx-cli-3.0
    drwxr-xr-x  14 hapresto  staff   448 Dec 18 09:09 cisco-nx-cli-5.13
    drwxr-xr-x  13 hapresto  staff   416 Nov 29 05:17 dell-ftos-cli-3.0
    drwxr-xr-x  10 hapresto  staff   320 Nov 29 05:17 juniper-junos-nc-3.0
    ```

    </details>

1. And now you have the newer NED versions available, as well as the demo/evaluation versions included with NSO itself! 


### `ncsrc`
The last thing we want to notice are the files `ncsrc` and `ncsrc.tsch`.  These are shell scripts for bash and tsch that setup your `PATH` and other environment variables for NSO.  Depending on your shell, you'll `source` this file before starting your NSO work.  

Many users will update their `~/.bash_profile` files to automatically `source ~/nso-5.3/ncsrc` for every terminal, or you can just do it manually when you want it.  Once it has been "sourced", you now have access to all the NSO executables - which start with `ncs`

```bash
ncs {TAB} {TAB}

# Output
ncs                      ncs-maapi                ncs-project              ncs-start-python-vm      ncs_cmd                  ncs_load                 
ncs-backup               ncs-make-package         ncs-setup                ncs-uninstall            ncs_conf_tool            ncsc                     
ncs-collect-tech-report  ncs-netsim               ncs-start-java-vm        ncs_cli                  ncs_crypto_keys 
```

We'll make use of some of these in our lab, but many of these are used for advanced topics and can be ignored for now.  

## Resources 
Here are some handy links to resources related to getting started with Cisco NSO.

* https://developer.cisco.com/site/nso/
* https://developer.cisco.com/docs/nso/#!getting-nso
* https://developer.cisco.com/docs/nso/#!an-nso-lab
* The "NSO Installation Guide" available in ${NCS_DIR}/doc 
