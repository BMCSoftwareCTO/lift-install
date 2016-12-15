# lift-install

## Installing BMC Lift
Prerequisite:
- DockerHub credentials and registered e-mail address are required to install Lift
- Centos 6.x or 7.x system with 16GB RAM

Run the following in a bash shell as root (substitute with appropriate values in <>, see sample below):

`export LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/<release tag>; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/<release tag>/liftinstall.sh | bash -s -- --reg_user "<DockerHub user>" --reg_password '<DockerHub password>' --reg_email "<DockerHub registered e-mail address>" --default_tag "<release tag>"`


### Installer Arguments

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
    --db_dir <arg>              // Default is /. If specified, lift and spinnaker db files will be
                                // created under the specified directory. The directory will be created
                                // if it doesn't exist. 
    --base_dir <arg>            // Default is standard /var/lib/... for Docker and Kubelet.  If specific, docker
                                // and kubelet storage will move to the specific directory.  This must be the same
                                // directory for all kuberenetes nodes and the master.   
    --ui_port <arg>             // Default is 8000. This is for spyglass client UI.
    --api_port <arg>            // Default is 9080. This is for API service.
    --aws_port <arg>            // Deprecated
    --ssh_port <arg>            // Deprecated
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
    --default_tag <arg>         // Default tag from DockerHub to pull for both lift and spinnaker. For Lift images,
                                // it can be overwritten by specific urls lift_<service>_url
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
    --aws_use_public_ip         // Default is false
                                // If passed in, only matters when installing on an aws instance.
                                // This flags whether or not to use
                                // the public or private ip of the aws system to do install with
    --dbnode true/false         // Deprecated.
    --lift_download_url <arg>   // Default: https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest
                                // Specifies where admiral will be downloading
                                // lift installer from for remote hosts.
    --disable_lift_oauth        // Default lift_oauth is enabled. By passing in this flag (no args needed
                                // after arg)
    --client_secret <secret>    // The client secret that is issued by the OAuth provider. Valid only if -auth_enabled is true
```

### Example: To install release tag 0.0.1-beta.2
```export LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/0.0.1-beta.2; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/0.0.1-beta.2/liftinstall.sh | bash -s -- --reg_user "<DockerHub user>" --reg_password '<DockerHub password>' --reg_email "<DockerHub registered e-mail address>" --default_tag 0.0.1-beta.2```

### Example: Installing BMC Lift with Optional Arguments using release tag latest
```export LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest/liftinstall.sh | bash -s -- --port_range "8000 9000" –ui_port 8081 –api_port 8099```

### Example: Installing BMC Lift NODE only
```export LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest/liftinstall.sh | bash -s -- --install_type NODE --master_hostname my-kube-master-host --sky_dns 10.254.167.114```

### Example: To see installer help
```export LIFT_DOWNLOAD_URL=https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest; wget -O - https://github.com/BMCSoftwareCTO/lift-install/releases/download/latest/liftinstall.sh  | bash -s -- --help```


### Installation Directory

`/opt/bmc/lift/installer`

## Configuring a Spinnaker Stack Outside of BMC Lift

In the event that a Spinnaker configuration is not supported by BMC Lift, a Spinnaker stack that was deployed and configured via Lift can be reconfigured manually by following these steps:

On machine where BMC Lift is installed perform the following steps:

* Retrieving Admiral container ID
  * Run `docker ps | grep admiral` and obtain the Admiral container ID
* Copy template files from Admiral container
  * Run `mkdir spkr-templates`
  * Run `cd spkr-templates`
  * Run `docker cp <Admiral container ID>:/opt/bmc/lift/kube/spinnaker/templates .`
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

## Manually Updating Lift Service Versions

Connect to the machine hosting the Lift application. Perform the following as root.

* `cd /opt/bmc/lift/installer.tmp/kubernetes/lift/rcs`
  * Make backups of *.yaml
    * lift-admiral-rc.yaml
    * lift-api-rc.yaml
    * lift-aws-rc.yaml
    * lift-oauth-rc.yaml
    * lift-ssh-rc.yaml
    * lift-cassandra-rc.yaml
    * lift-spyglass-rc.yaml
  * Get existing Docker images versions
    * `curl https://github.com/BMCSoftwareCTO/lift-install/releases/download/lift-versions/snapshots.json`
    * Each entry looks like:
    ```
			    {
			        "0.0.1-sprint.30": {
			            "api": "bmcsoftware/lift-api:0.0.1-sprint.30",
			            "oauth": "bmcsoftware/lift-oauth-server:0.0.1-sprint.30",
			            "admiral": "bmcsoftware/lift-admiral:0.0.1-sprint.30",
			            "spyglass": "bmcsoftware/lift-spyglass:0.0.1-sprint.30",
			            "aws": "bmcsoftware/lift-aws:0.0.1-sprint.30",
			            "ssh": "bmcsoftware/lift-ssh:0.0.1-sprint.30",
			            "cassandra": "bmcsoftware/lift-cassandra:0.0.1-sprint.30"
			        },
			        "timestamp": 1477517811620
			    }
    ```
  * Edit each lift-XXX-rc.yaml file and replace the "image" property with the Docker image path desired
    * e.g. for lift-aws-rc.yaml
      * From
        * `image: bmcsoftware/lift-aws:latest`
      * To
        * `image: bmcsoftware/lift-aws:0.0.1-sprint.30`
  * Use kubectl to delete each of the Lift Kubernetes replication controllers (rc)
    * e.g.
      * `kubectl delete rc/aws --namespace=bmclift-ns`
  * Create the rcs from the lift-XXX-rc.yaml files
    * e.g.
      * `kubectl create -f lift-aws-rc.yaml`
      
Kubernetes will pull the new path for the Docker images for each of the Lift rcs

## Manually Updating Spinnaker Service Versions

* Get admiral CLUSTER-IP
  * kubectl get svc --namespace=bmclift-ns
    * e.g.
    ```
		NAME                CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
		svc/admiral-svc     10.254.115.22    <none>        9083/TCP   4h
		svc/api-svc         10.254.146.148   <nodes>       9080/TCP   4h
		svc/aws-svc         10.254.247.5     <none>        9081/TCP   4h
		svc/cassandra-svc   10.254.75.236    <none>        9042/TCP   4h
		svc/oauth-svc       10.254.126.122   <nodes>       8080/TCP   4h
		svc/spyglass-svc    10.254.39.138    <nodes>       8000/TCP   4h
		svc/ssh-svc         10.254.155.21    <none>        9082/TCP   4h
    ```
* List Spinnaker instances to a file
  * e.g.
    * `curl http://10.254.115.22:9083/v1/spinnakers >spinnakers.json`
    * Use a JSON-pretty tool to make it readable
      * e.g.
      ```
			[
			  {
			    "id": "11bcc9a0-9c84-11e6-8fab-316805b20aec",
			    "name": "192.168.217.129",
			    "status": "Failed",
			    "size": "small",
			    "deckurl": "http://192.168.217.129:9000",
			    "providers": [
			      "11abb2a0-9c84-11e6-8fab-316805b20aec",
			      "ef4d05b0-9c83-11e6-8fab-316805b20aec"
			    ],
			    "accounts": [
			      "11ac27d0-9c84-11e6-8fab-316805b20aec",
			      "ef4c6970-9c83-11e6-8fab-316805b20aec"
			    ],
			    "credentials": null,
			    "hosts": [
			      "10d576e0-9c84-11e6-8fab-316805b20aec"
			    ],
			    "integrations": null,
			    "notifications": null,
			    "authentications": null,
			    "deckHost": "192.168.217.129",
			    "namespace": "spinnaker-192168217129-1477600173141",
			    "gatePort": "8084",
			    "deckPort": "9000",
			    "roscoPort": "8087"
			  }
			]
      ```
* Pick Spinnaker instance id where the name attribute matches the Spinnaker stack whose Docker images are to be updated
  * e.g.
    * ```"id": "11bcc9a0-9c84-11e6-8fab-316805b20aec",```
* Get Lift admiral Docker image path
  * `kubectl get po --namespace=bmclift-ns | grep admiral`
    * e.g.
        ```admiral-z0tnp     1/1       Running   0          2h```
* `kubectl get po/admiral-z0tnp -o jsonpath={.spec.containers[*].image} --namespace=bmclift-ns`
  * e.g.
        ```192.168.217.129:5000/cholin/lift-admiral:latest```
* Copy Spinnaker rc files for Spinnaker stack from admiral Docker container to current directory (tmp)
  * `docker cp $(docker ps | grep 192.168.217.129:5000/cholin/lift-admiral:latest | awk '{print $1;}'):/opt/bmc/lift/kube/spinnaker/templates/11bcc9a0-9c84-11e6-8fab-316805b20aec/rcs/. .`
* Get manifest of Spinnaker Docker image versions
  * `curl https://github.com/BMCSoftwareCTO/lift-install/releases/download/spkr-versions/snapshots.json`
  * Each entry looks like:
```
        {
            "0.0.1-sprint.30": {
                "clouddriver": "bmcsoftware/spinnaker-clouddriver:0.0.1-sprint.30",
                "deck": "bmcsoftware/spinnaker-deck:0.0.1-sprint.30",
                "echo": "bmcsoftware/spinnaker-echo:0.0.1-sprint.30",
                "front50": "bmcsoftware/spinnaker-front50:0.0.1-sprint.30",
                "gate": "bmcsoftware/spinnaker-gate:0.0.1-sprint.30",
                "igor": "bmcsoftware/spinnaker-igor:0.0.1-sprint.30",
                "orca": "bmcsoftware/spinnaker-orca:0.0.1-sprint.30",
                "rosco": "bmcsoftware/spinnaker-rosco:0.0.1-sprint.30"
            },
            "timestamp": 1477519302868
        }
 ```
* For each Spinnaker service whose Docker image is to be updated
  * Edit spkr-XXX.yaml and replace the "image" property with the Docker image path desired
    * e.g. for spkr-igor.yaml
      * From
		```image: bmcsoftware/spinnaker-igor:latest```
      * To
		```image: bmcsoftware/spinnaker-igor:0.0.1-sprint.30```
  * Use kubectl to delete each of the Spinnaker Kubernetes replication controllers (rc)
    * e.g.
      * Get namespace from spkr-igor.yaml file in the "namespace" field
        * e.g.
          * ```namespace: spinnaker-192168217129-1477600173141```
      * Delete existing Spinnaker service rc
        * ```kubectl delete rc/spkr-igor-v000 --namespace=spinnaker-192168217129-1477600173141```
  * Create the rcs from the lift-XXX-rc.yaml files
    * e.g.
      * ```kubectl create -f spkr-igor.yaml```
    * Kubernetes will pull the new path for the Docker images for the Spinnaker service
* Now make the Docker image path updates permanent so that the next Spinnaker stack update or new Spinnaker stack instance will use the desired Spinnaker Docker images
  * Copy Spinnaker template yamls from Admiral container to host file system
  ```
    mkdir -p /opt/bmc/lift/installer.tmp/kubernetes/spinnaker/templates/rcs`
	docker cp $(docker ps | grep 192.168.217.129:5000/cholin/lift-admiral:latest | awk '{print $1;}'):/opt/bmc/lift/kube/spinnaker/templates/rcs/. /opt/bmc/lift/installer.tmp/kubernetes/spinnaker/templates/rcs/.
  ```
  * Edit the rc yamls image paths in /opt/bmc/lift/installer.tmp/kubernetes/spinnaker/templates/rcs/ as before
  * Edit the Lift Admiral rc yaml file to mount the updated Spinnaker template rc yaml files
	`/opt/bmc/lift/installer.tmp/kubernetes/lift/rcs/lift-admiral-rc.yaml`
    * Under "volumes:" add
    ```
      -
        name: spkr-template-rcs
        hostPath:
          path:	/opt/bmc/lift/installer.tmp/kubernetes/spinnaker/templates/rcs
    ```
    * Under "volumeMounts:" add
    ```
    -
      name: spkr-template-rcs
      mountPath: /opt/bmc/lift/kube/spinnaker/templates/rcs
    ```
  * Use kubectl to delete each of the Lift Kubernetes replication controller for Admimral (rc)
    * e.g.
  ```kubectl delete rc/admiral --namespace=bmclift-ns```
  * Create the rcs from the lift-admiral-rc.yaml files
    * e.g.
  ```kubectl create -f lift-admiral-rc.yaml```
