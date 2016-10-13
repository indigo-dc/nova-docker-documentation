# Installation and configuration manual

This section describes how to install and configure the nova-docker as a Nova
Compute node.

## Pre-requisites

To install a nova-docker compute node, you should have fully functional OpenStack
infrastructure:
* [Openstack install guide](http://docs.openstack.org/index.html#install-guides)

Supported OpenStack versions are Liberty and Mitaka.

The compute node should be installed and configured according to the Openstack
installation manual referred previously, namely the nova compute and neutron
services.

Install docker-engine and start the service. The installation and configuration
instructions can be in [Install Docker Engine](https://docs.docker.com/engine/installation/).

Check that the docker service is running:

```bash
systemctl status docker
```

Additional or modified configuration for the nova-docker steps are explained in
the sections below.

As nova needs to spawn docker containers, the nova user must be in the docker group; hence execute the command:
```bash
usermod -aG docker nova
```

## Installation

Indigo-DataCloud 1st release components are supported in Centos7 and Ubuntu 14.04.

The next sections detail installation instruction for each Operating System.

### Installation on Ubuntu 14.04

The first step is to install the Indigo-DataCloud gpg keys and apt package
repositories. Further details can be found [here](https://indigo-dc.gitbooks.io/indigo-datacloud-releases/content/chapter1.html)

All packages are signed with the INDIGO - DataCloud gpg key. Import the public
key:

```bash
wget -q -O - http://repo.indigo-datacloud.eu/repository/RPM-GPG-KEY-indigodc | \
sudo apt-key add -
```

Install the apt repository:

```bash
wget http://repo.indigo-datacloud.eu/repository/indigo/1/ubuntu/dists/trusty/main/binary-amd64/indigodc-release_1.0.0-1_amd64.deb

dpkg -i indigodc-release_1.0.0-1_amd64.deb
```

Install the nova-docker package:

```bash
apt-get -y install python-nova-docker
```

### Installation on Centos7

The first steps are the same as for Ubuntu 14.04, that is import the gpg key

```bash
rpm --import http://repo.indigo-datacloud.eu/repository/RPM-GPG-KEY-indigodc
```

Install the indigo1 repository for Centos7

```bash
yum -y install http://repo.indigo-datacloud.eu/repository/indigo/1/centos7/x86_64/base/indigodc-release-1.0.0-1.el7.centos.noarch.rpm
```

Install the nova-docker package:

```bash
yum -y install python-nova-docker
```

## Configuration

### Nova compute machine

Nova needs to be configured to use the Docker virt driver.
Edit the configuration file /etc/nova/nova.conf according to the following options:

```
[DEFAULT]
compute_driver = novadocker.virt.docker.DockerDriver
```

Create the directory /etc/nova/rootwrap.d, if it does not
already exist, and inside that directory create a file "docker.filters" with the following content:

```
# nova-rootwrap command filters for setting up network in the docker driver
# This file should be owned by (and only-writeable by) the root user
[Filters]
# nova/virt/docker/driver.py: 'ln', '-sf', '/var/run/netns/.*'
ln: CommandFilter, /bin/ln, root
```

Restart the nova compute service

```bash
systemctl restart openstack-nova-compute
```

### Glance service

In the machine where you are running the **glance service**, it needs to be
configured to support the docker container format.

It's important to leave the default ones in order to not break an existing
glance install.

```
[DEFAULT]
container_formats = ami,ari,aki,bare,ovf,docker
```

Restart the glance services

```bash
systemctl restart openstack-glance-api openstack-glance-registry
```
