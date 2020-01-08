# Cleaning Up Your Environment
So you've finished up the exercises, and want to cleanup the lab.  It's super easy, let's get to it!  

## Stopping and Deleting the NSO Instance

> **Super Simple Shortcut**: In the [Makefile](nso-instance/Makefile), there is a target for `make clean` that will stop and delete the NSO instance for you by running the same commands we describe below.  

1. Jump into the `nso-instance` directory if you aren't already there.  
1. First step is to stop the running instance of NSO.  

    ```
    ncs --stop
    ```

    > There won't be any output, but the command takes a few seconds to complete. 

1. Now you just need to delete the files in the directory for NSO.  Unfortunately there isn't a `ncs-cleanup` command so we need to do it manually.  

    ```
    rm -Rf README.ncs \
        state/ \
        logs/ \
        ncs-cdb/ \
        target/ \
        packages/ \
        scripts/ \
        ncs.conf \
        storedstate \
        SAEventRegular*
    ```

    * The above directories and files hold the running configuration, logs, and CDB for the NSO instance we've been using.  Here are some highlights on them. 
        * `state/` - directory holding active "state" for the NSO application.  Things like compliance reports, active NEDs and code, events, etc. 
        * `logs/` - big shocker here, but this directory holds most of the log files for NSO.  As you work with NSO more and need to troubleshoot something this directory will become a gold mine.  
        * `ncs-cdb\` - the Configuration Database for NSO.  The files are an internal format so you can't read them directly.  
        * `packages\` - the NEDs and other packages/services that are loaded up when NSO starts.  *Often this will be symlinks to the NSO installation directory.*
        * `ncs.conf` - the XML based configuration file for NSO.  This file controls things like how authentication is handled, directory locations for key files nad logs, and more.  As you progress with NSO more, you'll use this file to make NSO more "your own". 
        * `SAEventRegular*` - log files that are NOT put into `logs\` related to licensing checks. 

That's it... you've now flushed out and cleared the lab instance of NSO and can start over again if you like.  

And next time around, rather than running the commands manually like that, you can be a super skilled programmer and simply do this... 

```bash
make clean

# Output
Stopping and Clearing NSO Instance...
```

## Tearing Down the Sample VIRL Network
If you've been using the included VIRL topology and DevNet Sandbox (and why wouldn't you be), you just need to stop the simulation.  Here we'll shut it down using the same `virlutils` tool we used to start it.  

1. Jump back up to the root of the `nso-getting-started` directory, and verify that your simulation is active.  

    ```bash
    # assuming you're in the 'nso-instance' directory
    cd ..

    virl ls
    ```

    <details><summary>Output</summary>

    ```
    Running Simulations
    ╒════════════════════════════════════╤══════════╤════════════════════════════╤═══════════╕
    │ Simulation                         │ Status   │ Launched                   │ Expires   │
    ╞════════════════════════════════════╪══════════╪════════════════════════════╪═══════════╡
    │ nso-getting-started_default_S034t6 │ ACTIVE   │ 2019-12-15T16:27:40.253312 │           │
    ╘════════════════════════════════════╧══════════╧════════════════════════════╧═══════════╛
    ```

    </details>

1. Shutdown the simulation. 

    ```bash
    virl down
    ```

    <details><summary>Output</summary>

    ```bash
    Removing ./.virl/default
    Shutting Down Simulation nso-getting-started_default_S034t6.....
    SUCCESS
    ```

    </details>