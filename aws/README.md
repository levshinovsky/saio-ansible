Purpose of this Git Repo
=================================
This repo provides ansible playbooks to build an SAIO system on
 * AWS ( Amazon Web Service / EC2 )

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
SAIO ansible playbook via AWS ( EC2 )
=================================
## AWS Preparation
You use access keys to sign programmatic requests that you make to AWS if you use the AWS SDKs, REST, or Query APIs. The AWS SDKs use your access keys to sign requests for you, so that you don't have to handle the signing process.

### Boto AWS Module for ansible
All of the modules require and are tested against recent versions of ***boto***. You’ll need this Python module installed on your control machine.

1. `sudo pip install boto`

### AWS Access keys (access key ID and secret access key) Setup
Original Reference from AWS Doc: [Managing Access Keys for your AWS Account](http://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html)
#### via Security Credentials - Creating, Disabling, and Deleting Access Keys for your AWS Account
1. Log in with your AWS Account and go to
   [aws](https://aws.amazon.com/) and click on the
   `My Account` --> `AWS Management Console`.
1. In the upper right of the console, choose the account name or number and then choose Security Credentials.
1. On the AWS Security Credentials page, expand the Access Keys (Access Key ID and Secret Access Key) section.
1. Choose Create New Access Key. You can have a maximum of two access keys (active or inactive) at a time.
1. Choose Download Key File to save the access key ID and secret access key to a .csv file on your computer. After you close the dialog box, you can't retrieve this secret access key again.
1. To disable an access key, choose Make Inactive. AWS denies requests signed with inactive access keys. To re-enable the key, choose Make Active.
1. To delete an access key, choose Delete. To confirm that the access key was deleted, look for Deleted in the Status column.

 * Caution: Before you delete an access key, make sure it is no longer in use. You can't recover a deleted access key.
 * Other than Security Credential, you can create Access/Secrety Key via [IAM](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html#Using_CreateAccessKey)
 * PS: Temporary Access Key: You can use temporary access keys in less secure environments or distribute them to grant users temporary access to resources in your AWS account

### AWS Key pairs (SSH) Setup
For Amazon EC2, you use key pairs to access Amazon EC2 instances, such as when you use SSH to log in to a Linux instance. [Amazon EC2 Key Pairs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)
#### [Creating Your Key Pair Using Amazon EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair)
1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
1. In the navigation pane ( Left-hand side EC2 Dashboard Menu ), under **NETWORK & SECURITY**, choose **Key Pairs**.

 * Tip: The navigation pane is on the left side of the Amazon EC2 console. If you do not see the pane, it might be minimized; choose the arrow to expand the pane.

1. Choose **Create Key Pair**.
1. Enter a name for the new key pair in the **Key pair name** field of the **Create Key Pair** dialog box, and then choose **Create**.
1. The private key file is automatically downloaded by your browser ( <key pair name>.pem ). The base file name is the name you specified as the name of your key pair, and the file name extension is .pem. Save the private key file in a safe place.

 * Important: This is the only chance for you to save the private key file. You'll need to provide the name of your key pair when you launch an instance and the corresponding private key each time you connect to the instance.

1. If you will use an SSH client on a Mac or Linux computer to connect to your Linux instance, use the following command to set the permissions of your private key file so that only you can read it.
`$ chmod 400 my-key-pair.pem`

### Or you can import your key pair
#### [Importing Your Own Key Pair to Amazon EC2](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#how-to-generate-your-own-key-and-import-it-to-aws)
1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
1. In the navigation pane ( Left-hand side EC2 Dashboard Menu ), under **NETWORK & SECURITY**, choose **Key Pairs**.
 * Tip: The navigation pane is on the left side of the Amazon EC2 console. If you do not see the pane, it might be minimized; choose the arrow to expand the pane.
1. Choose **Import Key Pair**.
1. In the **Import Key Pair** dialog box, choose **Browse**, and select the public key file that you saved previously. Enter a name for the key pair in the Key pair name field, and choose **Import**.

### Sum of AWS Preparation
At this point you should have
 * Install boto
 * Access Key ID:
 * Secret Access Key:
 * create or import SSH key
   * Create SSH ( Public Key in AWS and Download Private Key xxx.pem )
   * Import your own Public Key and keep your own private key in ~/.ssh/

## Build and Test
### Boot RHEL7 VM (AWS doesn't provide official CentOS-7)
1. `cd aws`
1. `ansible-playbook aws_create.yml`
1. find out external ip in AWS Management Console -> EC2 -> Select VM --> Description Tab Or from ansible output
```
TASK [Add all instance public IPs to host group] *******************************
[DEPRECATION WARNING]: Using bare variables is deprecated. Update your playbooks so that the environment value uses the full variable syntax
('{{ec2.instances}}').
This feature will be removed in a future release. Deprecation warnings can be disabled by setting deprecation_warnings=False in
ansible.cfg.
changed: [localhost] => (item={u'kernel': None, u'root_device_type': u'ebs', u'private_dns_name': u'ip-xxxx.us-west-1.compute.internal', u'public_ip': u'xxxx', u'private_ip': u'xxxx', u'id': u'i-5b67c4cc', u'ebs_optimized': False, u'state': u'running', u'virtualization_type': u'hvm', u'architecture': u'x86_64', u'ramdisk': None, u'block_device_mapping': {u'/dev/sda1': {u'status': u'attached', u'delete_on_termination': True, u'volume_id': u'vol-ac4c7902'}}, u'key_name': u'johnnywa', u'image_id': u'ami-d1315fb1', u'tenancy': u'default', u'groups': {u'sg-160e2572': u'default'}, u'public_dns_name': u'ec2-xxxx.us-west-1.compute.amazonaws.com', u'state_code': 16, u'tags': {u'Name': u'foo'}, u'placement': u'us-west-1a', u'ami_launch_index': u'0', u'dns_name': u'ec2-xxxx.us-west-1.compute.amazonaws.com', u'region': u'us-west-1', u'launch_time': u'2016-10-04T06:18:41.000Z', u'instance_type': u't2.medium', u'root_device_name': u'/dev/sda1', u'hypervisor': u'xen'})

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=2    unreachable=0    failed=0
```

### Build SAIO

#### AWS Build Output Screenshot
 * https://gist.github.com/chianingwang/047f9035559c5119d80c0c3011d9c962

#### Install SAIO on AWS EC2 VM
1. `cd ..` Back to root directory
1. `ansible-playbook site.yml -i "<gce external ip>,"`

### SSH to the Box
1. `ssh <aws external ip>`

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
### Wipe Out AWS VM
1. `cd aws`
1. `ansible-playbook aws_delete.yml`
