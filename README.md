# OpenStack nova-docker

## Overview

## Setup

Pre-requisites:

Have a complete Openstack infrastructure
Setup a new Compute Node

Done on Centos7 with Openstack Kilo
Deploy a nova compute node to a pre-existing installation:
Installed nova, neutron, openvswitch
wget the rpm and installed it

```
# yum install docker-engine-1.8.1-1.el7.centos.x86_64.rpm
```

Enable and start service

```
[root@comp ~]# systemctl enable docker
[root@comp ~]# systemctl start docker
```

Get nova-docker from github

```
# git clone https://github.com/stackforge/nova-docker.git -b stable/kilo
# pip install -e /root/nova-docker
```

Nova configuration
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

Glance configuration

Glance needs to be configured to support the "docker" container format.
It's important to leave the default ones in order to not break an existing glance install.

```
[DEFAULT]
container_formats = ami,ari,aki,bare,ovf,docker
```

## Usage

To register a docker image in glance, first pull one from the docker registry
$ docker pull centos

After you can create the image in glance
$ docker save centos| glance image-create --is-public=True --container-format=docker --disk-format=raw --name centos

## Notes and tests

nova-docker does not launch images in an interactive way (i.e. it doesn't invoke "docker run -t -i", but presumably only "docker run -t"). Then, in order to use interactively a docker-based container, a program accepting user connections (via ssh) must be launched at container startup (see this as an example: https://github.com/indigo-dc/CentOS-SSH-Access).
Docker public registry already has some image that can span interactive container: fedora/ssh is only one example, but there is more.
When saving images into glance, the name specified must exactly match the name as reported by "docker images".
"Docker images needs to be saved in glance in the same name as it is listed in the ‘docker images’ command, because the nova docker will get the image name from glance and will actually use it from the image saved in the docker, mismatch in names will cause failure in launching docker containers." (https://goo.gl/CgLJNI)
In order to have an heterogeneous cloud (hypervisors with KVM and others with nova-docker) different host aggregates must be created to contain docker hypervisors and KVM ones, and metadata must be set for them; for example:
"mymetadata=xyzzy" for aggregates containing docker hypervisors
"mymetadata=12345" for aggregates that containing libvirt/kvm hypervisors
The same metadata must be set to dedicated flavors, i.e.
"mymetadata=xyzzy" for docker flavors
"mymetadata=12345" for "legacy" flavors
and the user must choose the proper flavor if he/she wants to launch a container or a VM; the scheduler does the job: it matches the flavor's metadata to the host aggregate's metadata and send the image to launch to the proper hypervisor.
This has been tested and seems to work correctly.
No volume(cinder) attach/detach support at runtime so far (http://goo.gl/J9y9oo) as mentioned above (there are related bugs)
Must be investigated more deeply if there's the possibility to build a special image to get support for interactive console (textual or GUI): https://bugs.launchpad.net/nova-docker/+bug/1321818
Hybrid hypervisor: in theory it is possible to configure a single hypervisor to run 2 instances of nova-compute, one with Novadocker driver, the other one with Libvirt(KVM) driver. But also 2 instances of neutron-openswitch-agent must be run on the same machine otherwise neutron-server will not be able to communicate with nova-compute to communicate information about VIF. I've tried this configuration (which is documented here: https://goo.gl/ChVscU) but the system is very unstable: running more than 3 or 4 container results in instances without network connections and then useless. In several places on the WEB I found that it is highly not recommended to run more than one instance of neutron ovs agent on the same machine.
Block device attaching at container's boot time: see log here: http://pastebin.com/RiELAxtc
In few words: With Docker 1.9 I can attach host's block devices to a container (and use it from within the container) only a boot time and adding the special capability SYS_ADMIN. I do not know if novadocker uses (or will use) the --cap-add option; if it doesn't and it will not, the bd attach and use is impossible (even with --privileged option, which I guess is not even supported by novadocker).
Quotas
The system refuses to start new containers if the number of CPUs, instances or RAM is exceeded. nova-docker does not interfere with the quotas system, and consumed resources are accounted properly.
