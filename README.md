# lift-install

## Installing BMC Lift

Run the following in a bash shell:

`LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/liftinstall.sh | bash`

### Optional Arguments

```
--install_type <value>                               // MASTER, NODE, BOTH  (default is ‘BOTH’)
                                                     // If selecting NODE, please provide other parameters: 
                                                     // master_hostname and sky_dns_ip
--master_hostname <value>                            // this is needed when the type is ‘NODE’
--sky_dns_ip <value>                                 // this is needed when the type is ‘NODE’
                                                     // This can be obtained from MASTET/BOTH installenv.out file.
                                                     // Look for "SKY_DNS_CLUSTER_IP" in the install log.
                                                     // It is typically located at /opt/bmc/lift/logs/
--port_range <min max>                               // this is when you specify port range for lift (default: 8000 9999)
--ui_port <value>                                    // this should fall with in the port_range (default: 8000)
--api_port <value>                                   // this should fall with in the port_range (default: 9080)
--aws_port <value>                                   // this should fall with in the port_range (default: 9081)
--help                                               // this will display help info 
```
### Example: Installing BMC Lift with Optional Arguments
```LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/liftinstall.sh | bash -s -- --port_range "8000 9000" –ui_port 8081 –api_port 8099```

### Example: Installing BMC Lift NODE only
```LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/v0.0.1-alpha/liftinstall.sh | bash -s -- --install_type NODE --master_hostname my-kube-master-host --sky_dns 10.254.167.114```

### Installation Directory

`/opt/bmc/lift/installer`

## Configuring a Spinnaker Stack Outside of BMC Lift

In the event that a Spinnaker configuration is not supported by BMC Lift, a Spinnaker stack that was deployed and configured via Lift can be reconfigured manually by following these steps:

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
