Purpose of this Git Repo
=================================
This repo provides ansible playbooks to build an SAIO system on
 * GCE ( Google Cloud Platform )

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

SAIO ansible playbook via GCE
=================================
## Google Cloud Preparation
### Google Cloud Platform Setup
Prior to using this plugin, you will first need to make sure you have a
Google Cloud Platform account, enable Google Compute Engine, and create a
Service Account for API Access.

1. Log in with your Google Account and go to
   [Google Cloud Platform](https://cloud.google.com) and click on the
   `Try it free` button.
1. Create a new project and remember to record the `Project ID`
1. Next, enable the
   [Google Compute Engine API](https://console.cloud.google.com/apis/api/compute_component/)
   for your project in the API console. If prompted, review and agree to the
   terms of service.
1. While still in the API Console, go to
   [Credentials subsection](https://console.cloud.google.com/apis/credentials),
   and click `Create credentials` -> `Service account key`. In the
   next dialog, create a new service account, select `JSON` key type and
   click `Create`.
1. Download the JSON private key and save this file in a secure
   and reliable location.  This key file will be used to authorize all API
   requests to Google Compute Engine.
1. Still on the same page, click on
   [Manage service accounts](https://console.cloud.google.com/permissions/serviceaccounts)
   link to go to IAM console. Copy the `Service account id` value of the service
   account you just selected. (it should end with `gserviceaccount.com`) You will
   need this email address and the location of the private key file to properly
   configure this Vagrant plugin.
1. Add the SSH key you're going to use to GCE Metadata in `Compute` ->
   `Compute Engine` -> `Metadata` section of the console, `SSH Keys` tab. (Read
   the [SSH Support](https://github.com/mitchellh/vagrant-google#ssh-support)
   readme section for more information.)

   PS: if you don't have public key check it out [Generate SSH Key](https://cloud.google.com/compute/docs/instances/connecting-to-instance#generatesshkeypair)

### Google Cloud Connection via Configuring Modules with secrets.py
 * If you don't want to specify above info in your Ansible code, you can create
   secrets.py into your $PYTHONPATH
1. Create a file secrets.py looking like following, and put it in some folder
   which is in your $PYTHONPATH:
```
GCE_PARAMS = ('i...@project.googleusercontent.com', '/path/to/project.json')
GCE_KEYWORD_PARAMS = {'project': 'project_id'}
```
1. Find out your $PYTHONPATH `python -c "import sys; print sys.path"`
 * for example : "/usr/local/lib/python2.7/dist-packages"
1. move your secrets.py to $PYTHONPATH `mv ./secrets.py /usr/local/lib/python2.7/dist-packages`

### Install apache-libcloud for ansible gce driver
Ansible contains modules for managing Google Compute Engine resources, including creating instances, controlling network access, working with persistent disks, and managing load balancers. Additionally, there is an inventory plugin that can automatically suck down all of your GCE instances into Ansible dynamic inventory, and create groups by tag and other properties.

1. The GCE modules all require the apache-libcloud module which you can install from pip: `sudo pip install apache-libcloud`

### In sum you should have 5 important info and task done as below.
 * Install apache-libcloud
 * google_project_id
 * google_client_email
 * google_json_key_location
 * added your new project-wide SSH key.

## Build and Test
### Boot Cent7 VM
1. `cd gce`
1. `ansible-playbook gce_create.yml`
1. find out external ip in Google Cloud Platform -> Compute Engine -> VM Instances
```
TASK [Wait for SSH to come up] *************************************************
[DEPRECATION WARNING]: Using bare variables is deprecated. Update your playbooks so that the environment value uses the full variable syntax
('{{gce.instance_data}}').
This feature will be removed in a future release. Deprecation warnings can be disabled by setting
deprecation_warnings=False in ansible.cfg.
ok: [127.0.0.1] => (item={u'status': u'RUNNING', u'network': u'default', u'zone': u'us-central1-a', u'tags': [], u'image': u'centos-7-v20160921', u'disks': [u'foo'], u'name': u'foo', u'public_ip': u'130.211.203.240', u'private_ip': u'10.128.0.2', u'machine_type': u'n1-standard-1', u'metadata': {}})
```

### Build SAIO
1. `cd ..` Back to root directory
1. change "johnnywa" to your user name which should match
 1. private key user name,
 1. ssh user with /home/<username>
 * in "**site.yml**" and "**global_vars.yml**"
1. `ansible-playbook site.yml -i "<gce external ip>,"`

#### GCE Build Output Screenshot
 * https://gist.github.com/chianingwang/9324671187713f41a0b4eee5c209def8

### SSH to the Box
1. `ssh <gce external ip>`

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

## Clean Up
### Wipe Out GCE VM
1. `cd gce`
1. `ansible-playbook gce_delete.yml`
