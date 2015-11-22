# openshift-v3-installation-guide


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
export ec2_instance_type='t2.medium'
export ec2_master_instance_type='t2.medium'
export ec2_infra_instance_type='t2.medium'
export ec2_node_instance_type='t2.medium'
export ec2_etcd_instance_type='t2.medium'
export ec2_image='ami-dc1c2b8e'
export ec2_region='ap-southeast-1'
export ec2_keypair='dir-fpt-openShift'
export ec2_security_groups="['launch-wizard-4']"
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

### Terminate OpenShift
```
$ bin/cluster terminate aws <cluster-name>
```
