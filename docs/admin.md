# Administration and user manual
Test of nova-docker on Liberty platform
=======================================

NGINX Simple WEB Server image:
------------------------------ 

* `docker pull nginx`
* `docker save nginx | glance image-create --container-format=docker --disk-format=raw --name nginx`

(Please remember that the argument of --name *MUST BE EQUAL* to the name of the related docker image)

* `nova boot --flavor m1.small --image nginx --key-name mykey --availability-zone nova:<DOCKER-ENABLED-COMPUTE-NODE> --nic net-id=<NETWORK-ID> test-nginx`

It's about a simple WEB Server which can be connected to:

* `ip netns exec qrouter-<PROPERNAMESPACE> curl http://192.168.1.10`

(you should get a HTML code produced by the remote nginx's server).

DISVIS Indigo Cloud image:
--------------------------
* `docker pull indigodatacloudapps/disvis`
* `docker save disvis | glance image-create --container-format=docker --disk-format=raw --name disvis`
* `nova boot --flavor m1.medium --image "indigodatacloudapps/disvis" --key-name mykey --availability-zone nova:<DOCKER-ENABLED-COMPUTE-NODE> --nic net-id=<NETWORK-ID> test-disvis`

Check that the DISVIS container is running by trying a connection to its sshd server:

* ip netns exec qrouter-<PROPERNAMESPACE> telnet 192.168.1.11 22

``Trying 192.168.1.11...
Connected to 192.168.1.11.
Escape character is '^]'.
SSH-2.0-OpenSSH_7.2p2 Ubuntu-4ubuntu2.1``
