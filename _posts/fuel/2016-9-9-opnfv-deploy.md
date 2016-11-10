---
layout: post
title: "opnfv deploy and usage"
description: "opnfv deploy and usage"
catagory: tech
tags: opnfv
---

# Fuel

## Dependency

```bash:
sudo apt-get install -y libvirt-bin qemu-kvm python-pip fuseiso mkisofs genisoimage ipmitool
sudo apt-get install -y python-dev libz-dev libxml2-dev libxslt-dev libyaml-dev
sudo apt-get install build-essential libssl-dev libffi-dev 
sudo pip install cryptography pyyaml netaddr paramiko lxml scp pycrypto ecdsa
```

For Cryptography's install, please [refer](http://stackoverflow.com/questions/22073516/failed-to-install-python-cryptography-package-with-pip-and-setup-py)

##Code

```bash:
git clone https://github.com/wuwenbin2/fuel-plugin-onos.git
git clone https://gerrit.onosproject.org/onos
git clone https://github.com/opennetworkinglab/onos
git clone git://git.openstack.org/openstack/fuel-plugin-onos
git clone https://gerrit.opnfv.org/gerrit/fuel
git clone https://github.com/wuwenbin2/OnosSystemTest.git onos
git clone https://gerrit.opnfv.org/gerrit/functest
git clone https://gerrit.opnfv.org/gerrit/securedlab
```

## Deploy

### virtual

#### script
```bash:
#!/bin/sh -e
workbench=/fuel_deploy
iso=opnfv-2016-08-27_07-58-33.iso
scenario=os-onos-sfc-ha
exec $workbench/fuel/ci/deploy.sh -b file://$workbench/fuel/deploy/config/ -l huawei -p pod1 -s $scenario -i file:///root/ISO/$iso -H -B pxebr -L /fuel_deploy/fuel-deploy-baremetal-huawei_pod6.log.tar.gz 2>&1 | tee out.log
```
#### modification

```bash:
sed -i 's/value: kvm/value: qemu/g' fuel/deploy/config/dea_base.yaml 
sed -i 's/value: kvm/value: qemu/g' dea_base.yaml 
```

### Baremetal

#### Network configuration

Master：
Eth0： pxe
Eth1： public
Eth2： ipmi 

```bash:
cat /etc/network/interfaces
auto lo
iface lo inet loopback

auto eth1
iface eth1 inet static
address 192.168.22.2
netmask 255.255.255.0
gateway 192.168.22.1


auto eth2
iface eth2 inet static
address 172.16.131.152
netmask 255.255.255.0
gateway 172.16.131.1

auto pxebr
iface pxebr inet static
bridge_ports eth0          
address  10.20.0.1
broadcast 10.20.0.255
netmask 255.255.255.0
```

pxebr的ip必须为10.20.0.1 

##### dha

```bash:
vi securedlab/labs/huawei/pod6/fuel/config/dha.yaml
nodes:
- id: 1
  pxeMac: E8:4D:D0:BA:63:95
  ipmiIp: 172.16.131.11
  ipmiUser: root
  ipmiPass: Huawei@123
- id: 2
  pxeMac: E8:4D:D0:BA:63:51
  ipmiIp: 172.16.131.12
  ipmiUser: root
  ipmiPass: Huawei@123
```

注释： 
adapter: 使用默认的ipmi 即可
pxeMc 是node节点pxe启动的mac （将来master通过这个口，加上10.20.0.* ip地址） 
ipmiIp：是node节点 管理口的ip

##### dea-pod

```bash:
enp2s0f0:
  - fuelweb_admin
  - management
  - storage
  - private
  enp2s0f1:
- public
```

修改node接口名
enp2s0f0：pxe启动口
enp2s0f1: 外网口

```bash:
transformation
  - action: add-port
    bridge: br-fw-admin
    name: enp2s0f0
  - action: add-port
    bridge: br-mgmt
    name: enp2s0f0.101
  - action: add-port
    bridge: br-storage
    name: enp2s0f0.102
  - action: add-port
    bridge: br-mesh
    name: enp2s0f0.103
  - action: add-port
    bridge: br-ex
    name: enp2s0f1
```

#### Command
```bash:
sudo /fuel_deploy/fuel/ci/deploy.sh -b file:///fuel_deploy/securedlab    -l huawei -p pod6 -s os-onos-nofeature-ha -i file:///root/ISO/opnfv-2016-08-23_03-58-27.iso -H -B pxebr  -L /fuel_deploy/fuel-deploy-baremetal-daily-master_1.log.tar.gz
```

#### Clean
```bash:
cd fuel/deploy
sudo python deploy.py -co -dea ../ci/config/dea.yaml -dha ../ci/config/dha.yaml 
```
# Apex

## Usage

```bash:
ssh stack@192.168.37.1
cd /home/stack
source stackrc
openstack server list
```

log node：

```bash:
ssh heat-admin@192.0.2.9
```
openrc file：overcloudrc

# Compass:

## deploy

Directory requirement： /home
./build

File that needs modification： network_onos.yml

evn variable:

```bash:
export OPENSTACK_VERSION=mitaka
export OS_VERSION=trusty
```

### command

```bash:
./deploy.sh --dha /home/compass4nfv/deploy/conf/vm_environment/os-onos-sfc-ha.yml --network /home/compass4nfv/deploy/conf/vm_environment/huawei-virtual1/network_onos.yml --iso-url file:///home/compass4nfv/work/building/compass.iso
```

### code 

```bash:
git clone https://gerrit.opnfv.org/gerrit/compass4nfv
```
## usage

master:

```bash:
ssh 10.1.0.12
```
controller:

```bash:
ssh 10.1.0.50 
cd  /opt
. admin-openrc.sh
```

# Juju 
## maas

```bash:
~/.juju/environments 
grep -rn “192.168.122.2”
~/juju/joid_0712/joid/ci$ ./02-maasdeploy.sh
```

## Deploy:

```bash:
~/juju/joid_0712/joid/ci$ ./deploy.sh -t ha -s onos -o mitaka
Usage:
juju status --format=tabular
juju ssh glance/0
```

node with ovs ： neutron-gateway， nodes-compute

```bash:
openstack catalog show network
+-----------+-------------------------------------------+
| Field     | Value                                     |
+-------------------+-----------------------------------+
| endpoints | Canonical                                 |
|           |   publicURL: http://192.168.122.17:9696   |
|           |   internalURL: http://192.168.122.17:9696 |
|           |   adminURL: http://192.168.122.17:9696    |
|           |                                           |
| name      | neutron                                   |
| type      | network                                   |  
+-----------+-------------------------------------------+
openstack catalog show nova
```




