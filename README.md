# openshift-v3-installation-guide
### Create AWS environment
#### Create VPC
#### Create Security Group
```
• 22    - ssh
• 80    - Web Apps
• 443   - Web Apps (https)
• 4789  - SDN / VXLAN
• 8443  - OpenShift Console
• 10250 - kubelet
```
#### Create keypair

### Install dependencies
```
### Start a VM with Centos 7 AMI
### Update EPEL
$ wget http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
$ rpm -ivh epel-release-7-5.noarch.rpm
### Install dependencies
$ yum install -y ansible python-boto pyOpenSSL
$ yum install -y git
```

### Install OpenShift
```
$ mkdir openshift-installation
$ cd openshift-installation
$ git clone https://github.com/openshift/openshift-ansible.git
$ cd openshift-ansible

### export env var
$ touch env-var
### save below texts into env-var file
export AWS_ACCESS_KEY_ID=<your_access_key>
export AWS_SECRET_ACCESS_KEY=<your_secret_key>
export ec2_instance_type='t2.medium'
export ec2_master_instance_type='t2.medium'
export ec2_infra_instance_type='t2.medium'
export ec2_node_instance_type='t2.medium'
export ec2_etcd_instance_type='t2.medium'
export ec2_image='ami-dc1c2b8e'
export ec2_region='ap-southeast-1'
export ec2_keypair='keypair_name'
export ec2_security_groups='security_group_name'
export ec2_vpc_subnet='subnet-de1c64bb'
export ec2_assign_public_ip='true'
export os_etcd_root_vol_size='20'
export os_etcd_root_vol_type='standard'
export os_etcd_vol_size='20'
export os_etcd_vol_type='standard'
export os_master_root_vol_size='20'
export os_master_root_vol_type='standard'
export os_node_root_vol_size='15'
export os_docker_vol_size='50'
export os_docker_vol_ephemeral='false'

$ source env-var
### Create Cluster
$ bin/cluster create aws <cluster-name>
```
Note: If no deployment type is specified, then the default is origin
### Configure OpenShift
SSH into OpenShift master instance, change to ROOT account.
#### Deploy a Registry
```
$ cd /etc/origin/master
$ oadm registry --config=admin.kubeconfig \
    --credentials=openshift-registry.kubeconfig
# Check if the Registry up and running
$ oc get pods
$ oc get svc
```
#### Deploy a Router
```
$ oadm router <router_name> --replicas=<number> \
    --credentials=openshift-router.kubeconfig \
    --service-account=router
```
#### Setting the application domain
```
$ vi /etc/origin/master/master-config.yaml
```
Edit domain name in `subdomain` parameter:
```
routingConfig:
  subdomain: <your_domain>
```

### Terminate OpenShift
```
$ bin/cluster terminate aws <cluster-name>
```
