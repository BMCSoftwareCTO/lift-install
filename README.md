# lift-install

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
