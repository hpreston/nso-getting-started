# Minimum Viable Use Case: Network Configuration Compliance
A common request from network engineers and leadership is an easy way to verify configuration compliance.  This could be just for internal reasons, or to support an audit.  Whatever the reason, it can be a very challenging and time-consuming task to undertake.  Let's see how NSO can help.  

## Use Case Description 
We would like to ensure that the following network features are configured as desired across all devices in our network.  

* DNS server configuration 
* Syslog server configuration 
* NTP configuration

> Note: The technique used in this example could be extended to any aspect of the configuration.  For this demonstration we scoped it down to a set of features. 

## Cisco NSO Features Used 
* Device templates 
* Compliance reports 

## Building the Use Case 
This use case has two parts.  
1. Building a device template that contains the desired configuration to test. 
2. Building a compliance report to will check the configuration of a group of devices against a template

### Step 1: Building the Device Template 

1. Log into NSO with `ncs_cli -C -u admin` and enter `config` mode. 
1. Create the device template with the desired configuration for features.  

    ```
    devices template COMPLIANCE-CHECK
    ned-id cisco-ios-cli-6.42
    config

    ip name-server name-server-list 208.67.222.222
    ip name-server name-server-list 208.67.220.220
    exit

    service timestamps log datetime localtime show-timezone year
    logging host ipv4 10.225.1.11
    exit

    ntp server peer-list 10.225.1.11
    exit

    exit
    exit


    ned-id cisco-nx-cli-5.13
    config

    ip name-server servers 208.67.222.222
    ip name-server servers 208.67.220.220

    logging timestamp milliseconds
    logging server 10.225.1.11 level 5
    exit

    ntp server 10.225.1.11
    exit

    exit
    exit

    ned-id cisco-asa-cli-6.7
    config
    dns domain-lookup mgmt
    dns server-group DefaultDNS
    name-server 208.67.222.222
    name-server 208.67.220.220
    exit 

    logging timestamp
    logging host mgmt 10.225.1.11
    exit

    ntp server 10.225.1.11
    exit

    exit
    exit
    ```

1. Return to `top` and review the `configuration` you've staged for this template. 

    ```
    top
    show configuration
    ```

    <details><summary>Output</summary>

    ```
    devices template COMPLIANCE-CHECK
     ned-id cisco-ios-cli-6.42
      config
       service timestamps log datetime localtime
       service timestamps log datetime show-timezone
       service timestamps log datetime year
       ip name-server name-server-list 208.67.222.222
       !
       ip name-server name-server-list 208.67.220.220
       !
       logging host ipv4 10.225.1.11
       !
       ntp server peer-list 10.225.1.11
       !
      !
     !
     ned-id cisco-nx-cli-5.13
      config
       ip name-server servers [ 208.67.222.222 208.67.220.220 ]
       ntp server 10.225.1.11
       !
       logging server 10.225.1.11
        level 5
       !
       logging timestamp milliseconds
      !
     !
     ned-id cisco-asa-cli-6.7
      config
       logging timestamp
       logging host mgmt 10.225.1.11
       !
       ntp server 10.225.1.11
       !
       dns domain-lookup mgmt
       !
       dns server-group DefaultDNS
        name-server [ 208.67.222.222 208.67.220.220 ]
    ```
    </details>

1. Commit this new template to NSO. 

    ```
    commit
    ```

### Step 2: Building the Compliance Report 
1. Compliance reports simply compare a given template to a group of devices.  So the configuration is pretty simple.  Go ahead and enter this in. 

    ```
    compliance reports report COMPLIANCE-CHECK
    compare-template COMPLIANCE-CHECK ALL
    ```

1. Commit this into NSO.  

    ```
    commit
    ```

1. Exit `config` mode.  

    ```
    end
    ```

## Executing the Use Case 
Everything we need is ready for us to test the network for compliance.  

1. From the "enable mode" of NSO, go ahead and run the compliance report. 

    ```
    compliance reports report COMPLIANCE-CHECK run
    ```

    <details><summary>Output</summary>

    ```
    id 1
    compliance-status violations
    info Checking 20 devices and no services
    location http://localhost:8080/compliance-reports/report_1_admin_1_2019-12-20T14:46:34:0.xml
    ```
    </details>

1. Okay, so we see there are `violations` and a URL to the report.  Some things about compliance reports.
    * Results can come in in `html`, `text`, or `xml` format - `xml` is the default 
    * Because the results are likely very "verbose" and not something the engineer likely wants to just see at the CLI, they are saved as a file located in the folder `state/compliance-reports`
    * NSO runs a web-server so these reports are available by navigating to the given URL, or you can find them by navigating the folders on your server
1. Let's generate the `text` and `html` versions of the report as well. 

    ```
    compliance reports report COMPLIANCE-CHECK run outformat text
    compliance reports report COMPLIANCE-CHECK run outformat html
    ```

1. We'll start out by looking at the `text` version of the file.  Use whatever method you like to open up the `text` version of the report located within the `state/compliance-reports` directory. 

    ```bash
    (std) ~/code/nso-getting-started/nso-instance $ ll state/compliance-reports/

    total 56
    -rw-r--r--  1 hapresto  staff   8.8K Dec 20 14:46 report_1_admin_1_2019-12-20T14:46:34:0.xml
    -rw-r--r--  1 hapresto  staff   6.4K Dec 20 14:49 report_2_admin_1_2019-12-20T14:49:41:0.text
    -rw-r--r--  1 hapresto  staff   7.2K Dec 20 14:49 report_3_admin_1_2019-12-20T14:49:54:0.html
    ```

1. The top of the report provides some details about the report instance, then you'll get a list of devices and their compliance status.  Here you can see that there are `Discrepancies` found in all our devices. 

    ```text
    Template discrepancies

    COMPLIANCE-CHECK

        Discrepancies in device
        dmz-fw01-1
        dmz-rtr02-1
        dmz-rtr02-2
        dmz-sw01-1
        dmz-sw01-2
        dmz-sw02-1
        dmz-sw02-2
        fw-inside-sw01
        fw-outside-sw01
        fw01
        fw02
        internet-rtr
        leaf01-1
        leaf01-2
        leaf02-1
        leaf02-2
        spine01-1
        spine01-2
        vm-sw01
        vm-sw02
    ```

1. In the `Details` section, you'll find a device by device breakdown of what the problems are.  This is shown in a `diff` format, where lines with a `+` indicate missing configuration.  If there were configurations on the devices that should **not** be there, they'd be indicated with `-`. 

    > Note: Even though there are different CLI device types being evaluated (IOS, NX, ASA), NSO standardizes the "native" configuration into a uniform data model, in Junos curly-bracket style, when showing the diff. 

    ```diff
    Device dmz-rtr02-1

    config {
        service {
            timestamps {
                log {
                    datetime {
    +                    localtime;
    +                    show-timezone;
    +                    year;
                    }
                }
            }
        }
        logging {
            host {
    +            ipv4 10.225.1.11 {
    +            }
            }
        }
        ntp {
            server {
    +            peer-list 10.225.1.11 {
    +            }
            }
        }
    }
    ```

1. Go ahead and open up the XML and HTML versions of the report and take a look.  The data is the same, just formated differently.  

## Bonus: Resolving Compliance Problems
An obvious next step after running a compliance report would be to resolve any problems.  NSO makes this very simple as you've already done all the work you need to do!  

1. Enter `config` mode. 

1. Apply the template to your devices. 

    ```
    devices device-group ALL apply-template template-name COMPLIANCE-CHECK 
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

1. Use these commands to see the proposed changes to the network to bring the devices in compliance using the "native" device-type CLI. 

    ```
    show configuration 
    commit dry-run outformat native
    ```

1. Commit the configuration, which will have NSO send the commands to the devices and make them compliant. 

    ```
    commit
    ```

1. Exit config mode and re-run the compliance report, so NSO verifies that what is on the device is in compliance with the defined template. 

    ```
    end
    compliance reports report COMPLIANCE-CHECK run outformat text
    ```

    <details><summary>Output</summary>

    ```
    id 4
    compliance-status no-violation   < --- Woot!
    info Checking 20 devices and no services
    location http://localhost:8080/compliance-reports/report_4_admin_0_2019-12-20T15:4:27:0.text
    ```
    </details>

1. Now we have `no-violations` and are in good shape.  A report file is generated that you can submit for audits.  
