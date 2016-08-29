# Overview

Container support under linux relies on built in kernel features - cgroups for
limitation and prioritization of resources and namespaces to isolate
applications regarding process trees, networking, user IDs and mounted
filesystems.

A commonly used abstraction over this functionality is LXC (Linux Containers)
which exposes simplified interfaces and libraries. Tools like Docker provide an
additional abstraction layer simplifying deployment and management of container
units, introducing the concepts of images and tags.

OpenStack provides container support via its compute service called Nova.
Drivers exist to create and manage containers both using LXC (with the libvirt
driver) and using Docker. In both cases containers are treated as lightweight
virtual machines, offering similar isolation to traditional VMs but using the
same kernel as the host.

This document provides information about installation 
