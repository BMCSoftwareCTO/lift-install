# lift-install

## Installing BMC Lift
Prerequisite:
- DockerHub credentials and registered e-mail address are required to install Lift.
- Centos 6.x or 7.x system with 16GB RAM

Run the following in a bash shell as root:

`wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest/liftinstall.sh | bash -s -- --reg_user <DockerHub user>" --reg_password "<DockerHub password>" --reg_email "<DockerHub registered e-mail address>"`

### Optional Arguments

```
    --help                      // this will display help info 
    --install_type <arg>        // Default is "BOTH". 
                                // NODE: Installs a kubernetes node. If selecting NODE, 
                                // please provide other parameters: master_hostname, 
                                // sky_dns_ip
    --master_hostname <arg>     // This is required when doing NODE installation
    --sky_dns_ip <arg>          // This is required when doing NODE installation. This 
                                // can be obtained from MASTER/BOTH installenv.out file.
                                // Look for "SKY_DNS_CLUSTER_IP" in the install log.
                                // Install log is typically located at /opt/bmc/lift/logs/
    --reg_user <arg>            // Docker registry user login for accessing BMC docker 
                                // images.
    --reg_password <arg>        // Docker registry user password for accessing BMC docker 
                                // images.
    --reg_email <arg>           // Docker registry user email.
    --dbdir <arg>               // Default is /. If specified, lift and spinnaker db files will be
                                // created under the specified directory. The directory will be created
                                // if it doesn't exist.    
    --ui_port <arg>             // Default is 8000. This is for spyglass client UI.
    --api_port <arg>            // Default is 9080. This is for admiral service.
    --aws_port <arg>            // Default is 9081. This is for aws service.
    --ssh_port <arg>            // Default is 9082. This is for ssh service.
    --port_range <arg>          // Default is "8000 9999"
                                // Used to configure kubernetes api service where NodePort is 
                                // used. This should be a string with min and max port range 
                                // separated by space
                                // Example: "8001 9001"
    --insecure_regs <arg>       // Internal usage. For specifying one or more insecure docker
                                // registries where the lift related docker images will be pulled
                                // from. It should be in this format:
                                // "<ip:port>,<ip2:port>..."
                                // Example: "52.27.155.9:5000,172.22.238.229:5000"
    --lift_aws_url <arg>        // Internal usage. For specifying aws service docker
                                // image url.
                                // Example: 
                                // "clm-aus-008240.bmc.com:5000/lift-dev/ssh:22"
    --lift_admiral_url <arg>    // Internal usage. For specifying admiral service docker
                                // image url.
    --lift_spyglass_url <arg>   // Internal usage. For specifying spyglass service docker
                                // image url.
    --lift_ssh_url <arg>        // Internal usage. For specifying ssh service docker
                                // image url.
    --lift_cassandra_url <arg>  // Internal usage. For specifying cassandra docker
                                // image url.
    --dbnode true/false         // Deprecated.
    --lift_download_url <arg>   // Default: https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest
                                // Specifies where admiral will be downloading
                                // lift installer from for remote hosts.
```
### Example: Installing BMC Lift with Optional Arguments
```wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest/liftinstall.sh | bash -s -- --port_range "8000 9000" –ui_port 8081 –api_port 8099```

### Example: Installing BMC Lift NODE only
```wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest/liftinstall.sh | bash -s -- --install_type NODE --master_hostname my-kube-master-host --sky_dns 10.254.167.114```

### Example: To see installer help
```wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest/liftinstall.sh  | bash -s -- --help```

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
