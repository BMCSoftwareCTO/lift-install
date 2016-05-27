# lift-install

## Installing BMC Lift

Run the following in a bash shell:

`LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/liftinstall.sh | bash`

### Optional Arguments

```
-install_type <value>                                // MASTER, NODE, BOTH  (default is ‘BOTH’)
--master_hostname <value>                            // this is needed when the type is ‘NODE’
--sky_dns_ip <value>                                 // this is needed when the type is ‘NODE’
--port_range <a string with space separated values>  // this is when you specify ports for UI/api for lift
--ui_port <value>
--api_port <value>
```
### Example: Installing BMC Lift with Optional Arguments
```LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/liftinstall.sh | bash -s -- --port_range "8000 9000" –ui_port 8081 –api_port 8099```

## Configuring a Spinnaker Stack Outside of BMC Lift

In the event that a Spinnaker configuration is not supported by BMC Lift, a Spinnaker stack that was deployed and configured via Lift can be reconfigured manually be following these steps:

On machine where BMC Lift is installed perform the following steps:

* Connect to Admiral container
  * Run `docker ps | grep admiral` and obtain the Admiral container ID
* Copy template files from Admiral container
  * Run `mkdir spkr-templates`
  * Run `cd spkr-templates`
  * Run `docker cp <Admiral container ID>/opt/bmc/lift/kube/spinnaker/templates .`
* Edit Spinnaker yaml configurations
  * `cd config`
  * Edit config yaml as needed
* Get Kubernetes update scripts
  * Make a temp directory
  * `git clone https://github.com/spinnaker/spinnaker.git` into the temp directory
  * `cp <spinnaker repo dir>/experimental/kubernetes/scripts spkr-templates/.`
  * Edit spkr-templates/scripts/cleanup-config.sh and comment out the below with '#' characters:
  
```
     CONF_DIR=../../config/
     for FILENAME in $CONF_DIR*.yml; do
       rm config/${FILENAME:${#CONF_DIR}}
     Done
```
* Update Spinnaker stack configuration
  * Run `cd spkr-templates`
  * Run `bash scripts/update-config.sh`
  * Run `bash scripts/update-spinnaker.sh`
