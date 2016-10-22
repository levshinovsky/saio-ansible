Purpose of this Git Repo
=================================
This repo provides ansible playbooks to build an SAIO system on
 * Local VirtualBox via Vagrant - default
 * GCE ( Google Cloud Platform ) - in gce subfolder
 * AWS ( Amazon Web Service / EC2 ) - in aws subfolder

1. Before start, Please make sure you do install ansible
`sudo pip install ansible`

1. all the ansible for trigger from local ( control node )
In your playbook, the following pattern for provisioning steps will be using:
```
- hosts: localhost
  connection: local
  gather_facts: False
  tasks:
    - ...
```
SAIO Ansible playbook via Vagrant
=================================
## Preparation
This project assumes you have Virtualbox and Vagrant.

 * https://www.virtualbox.org/wiki/Downloads
 * http://www.vagrantup.com/downloads.html

- PS: I use VirtualBox 5.1.6 r110634 and Vagrant 1.8.5

## Build and Test
### Boot Cent7 and Build SAIO
1. `vagrant up`

### SSH to the Box
1. `vagrant ssh`

### Test Swift
1. `swift stat -v`
1. `echo 'this is test \n this is test \n this is test \n this is test \n this is test \n ' > test.txt`
1. `swift upload test_container test.txt`
1. `swift list `
1. `swift list test_container`

#### Double Check
1. `swift stat -v`

### Back to Local
1. `exit`

## Clean up
### Vagrant Destroy
1. `vagrant destroy`
