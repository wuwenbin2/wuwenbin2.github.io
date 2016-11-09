---
layout: post
title: "opnfv functest"
description: "opnfv functest"
category: tech
tags: [opnfv]
---

#Python flake8 
```bash:
apt-get install python-flake8
flake8 ./ 
```

#Functest

## Docker config 

1. install docker
2. pull docker image

```bash:
docker pull opnfv/functest
```

###Fuel
1. if it is virtual deployment  

```bash:
sudo ip link add name fuel1.101 link fuel1 type vlan id 101
sudo ip link set fuel1.101 up
sudo ip addr add 192.168.0.250/24 dev fuel1.101
```

2. launch container  

```bash:
sudo docker run --privileged=true -id -e INSTALLER_TYPE=fuel -e INSTALLER_IP=10.20.0.2     -e NODE_NAME=ericsson-pod2 -e DEPLOY_SCENARIO=os-onos-sfc-ha     -e BUILD_TAG=jenkins-functest-fuel-baremetal-daily-master-66 -e CI_DEBUG=false -v /home/jenkins/opnfv/functest/results/master:/home/opnfv/functest/results   opnfv/functest:latest /bin/bash
```

3. check  

```bash:
docker exec container_id ping 172.16.0.3
docker exec container_id ping 192.168.0.7
docker exec contailer_id ping 10.20.0.2
```   
   if 172.16.0.3 is unreachable, modify docker configuraion
   
```bash:
   vi /etc/default/docker
   DOCKER_OPTS="-b fuel4"
   
   service docker restart
```

  if 10.20.0.2 is unreachable, get openrc and place it in /root, then use command to copy credits into container.
  Or you can log in the container and scp the file to the correct location /home/opnfv/functest/conf/openstack.creds:
  
```bash:
sudo docker run --privileged=true -id -e INSTALLER_TYPE=fuel -e INSTALLER_IP=10.20.0.2     -e NODE_NAME=ericsson-pod2 -e DEPLOY_SCENARIO=os-onos-sfc-ha     -e BUILD_TAG=jenkins-functest-fuel-baremetal-daily-master-66 -e CI_DEBUG=false -v /root/openrc:/home/opnfv/functest/conf/openstack.creds   -v /home/jenkins/opnfv/functest/results/master:/home/opnfv/functest/results   opnfv/functest:latest /bin/bash
```

### Joid

launch container

```bash:
sudo docker run --privileged=true -id -e INSTALLER_TYPE=joid -e INSTALLER_IP=127.0.0.1    -e NODE_NAME=intel-pod6 -e DEPLOY_SCENARIO=os-onos-sfc-ha     -e BUILD_TAG=jenkins-functest-joid-baremetal-daily-master-81 -e CI_DEBUG=false -v /home/ubuntu/juju0710/joid/ci/admin-openrc:/home/opnfv/functest/conf/openstack.creds  -v /home/jenkins/opnfv/functest/results/master:/home/opnfv/functest/results   opnfv/functest:latest /bin/bash
```

### Compass
launch container

```bash:
sudo docker run --privileged=true -id -e INSTALLER_TYPE=compass -e INSTALLER_IP=127.0.0.1   -e NODE_NAME=huawei-pod1 -e DEPLOY_SCENARIO=os-onos-sfc-ha     -e BUILD_TAG=jenkins-functest-compass-baremetal-daily-master-120 -e CI_DEBUG=false   -v /home/jenkins/opnfv/functest/results/master:/home/opnfv/functest/results   opnfv/functest:latest /bin/bash
```

### Apex

* get Installer IP  
ssh 192.168.37.1 and get ip of eth0 for INSTALLER_IP

* launch container with replaced INSTALLER_IP  

```bash:
sudo docker run --privileged=true -id -e INSTALLER_TYPE=apex -e INSTALLER_IP=192.168.122.87 -e NODE_NAME=lf-pod1 -e DEPLOY_SCENARIO=os-onos-sfc-ha     -e BUILD_TAG=jenkins-functest-apex-apex-daily-master-daily-master-70 -e CI_DEBUG=false  -v /root/.ssh/id_rsa:/root/.ssh/id_rsa -v /home/jenkins-ci/opnfv/functest/results/master:/home/opnfv/functest/results  -v /home/jenkins-ci/stackrc:/home/opnfv/functest/conf/stackrc opnfv/functest:latest /bin/bash
```
* check network connectivity  
If 192.168.37.1 and 192.168.122.87 is unreachable, resolution:

```bash:
vi /usr/lib/systemd/system/docker.service
ExecStart=/bin/sh -c '/usr/bin/docker-current daemon -b br-public
systemctl daemon-reload
```
or 

```bash:
vi /etc/sysconfig/docker
='--selinux-enabled --log-driver=journald -b br-public'
```
Maybe it should be br-admin instead of br-public

* realod docker  

```bash:
docker daemon
```

## Run tests

```bash:
docker exec 759ff2468d /home/opnfv/repos/functest/ci/exec_test.sh -t onos
docker exec 759ff2468d /home/opnfv/repos/functest/ci/exec_test.sh -t onos_sfc
```
or 

```bash:
docker exec 759ff2468d  python /home/opnfv/repos/functest/ci/run_tests.py -t onos_sfc
docker exec 759ff2468d python /home/opnfv/repos/functest/ci/run_tests.py -t onos
docker exec 759ff2468d python /home/opnfv/repos/functest/ci/run_tests.py -t tempest_smoke_serial
```

#Yardstick


+ install docker  
For ubuntu: apt-get install docker.io  
For centos(apex): yum -y install docker  

+ pull docker image  

```bash:
docker pull opnfv/functest
```

+ launch a container  

```bash:
sudo docker run --privileged=true -id -e INSTALLER_TYPE=fuel -e INSTALLER_IP=10.20.0.2     -e NODE_NAME=ericsson-pod2 -e EXTERNAL_NETWORK=admin_floating_net     -e YARDSTICK_BRANCH=master -e DEPLOY_SCENARIO=os-onos-nofeature-ha   opnfv/yardstick  /bin/bash
```

+ run tests  

```bash:
docker exec id exec_tests.sh opnfv_os-onos-sfc-ha_daily.yaml
docker exec id exec_tests.sh opnfv_os-onos-nofeature-ha_daily.yaml
```

PS. You can refer commands from opnfv ci.


