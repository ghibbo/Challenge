# Challenge
This playbook provision 2 VM Centos, VMs ar local.
VMs are partitioned for ensuance that Docker has at least 40GB of available space.
Docker must to securely expose the Docker Daemon REST APIs.
Docker Daemon is configured as a service that starts automatically at system startup
Set up a Docker Swarm on the VMs, which is securely accessible.
Interaction and deploy services on the Swarm from local machine. 

![](avatar.png)

## Provisioning
Vagrant supports provisioning systems with Ansible.
This challenge uses the Ansible local provisioner. 
Vagrant will handle downloading and installing Ansible on the virtual guest, 
and then run a specified playbook locally on that virtual guest. 

## Check pre-requirements test lab locally
......

Inventory File
------------

If you want to use this role ......


Role Variables
--------------

