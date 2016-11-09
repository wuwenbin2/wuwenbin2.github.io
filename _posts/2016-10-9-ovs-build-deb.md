---
layout: post
title: "build ovs deb for sfc "
description: "ovs"
catagory: tech
tags: virtualization
---
+ dependency:
```bash
apt-get install autoconf     -y
apt-get install automake    -y
apt-get install libtool    -y
apt-get install make    -y
apt-get install graphviz debhelper dh-autoreconf libssl-dev python-all  python-qt4 python-twisted-conch   -y
apt-get install dpkg-dev    
```
+ build：

```bash
git clone https://github.com/yyang13/ovs_nsh_patches.git
git clone https://github.com/openvswitch/ovs.git
cd ovs
git reset --hard 7d433ae57ebb90cd68e8fa948a096f619ac4e2d8
cp ../ovs_nsh_patches/*.patch ./
git config user.email "you@example.com"
git config user.name "Your Name"
git am *.patch
cd ovs
./boot.sh;
./configure --with-linux=/lib/modules/`uname -r`/build;
make dist;
tar -xzf openvswitch-2.5.90.tar.gz
cd openvswitch-2.5.90;
dpkg-checkbuilddeps;DEB_BUILD_OPTIONS='parallel=8 nocheck' fakeroot debian/rules binary
```
