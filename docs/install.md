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

Additional or modified configuration for the nova-docker steps are explained in
the sections below.

## installation

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
