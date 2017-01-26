Ansible Playbook skeleton
=========================

# Description
Empty project to be used as starting point for an ansible playbook

# AWS EC2 dynamic inventory

### Introduction
Dynamic inventory script files taken from [here](https://github.com/ansible/ansible/tree/devel/contrib/inventory)
is included in folder _dynamicInventory_. The _ec2.py_ is the main script, _ec2.ini_ the setting file for the script.
The script should be used with public or private network, it depends on configuration.
To show the output of the script:
```
python dynamicInventory/ec2.py --refresh-cache --profile boto_profile_name
```
it will print the json of ec2 inventory, forcing the cache clean with option _--refresh-cache_

### Use with ansible
The _ec2.ini_ file is well documented inline. Here some useful option:
* __regions = eu-west-1__: filter the region
* __destination_variable = public_dns_name__ and __vpc_destination_variable = ip_address__: use these configurations to work with public IP
* __destination_variable = private_dns_name__ and __vpc_destination_variable = private_ip_address__: use these to work with private IP; usefel when the ansible configuration server is in the same VPC or when a bastion host is used.
* __instance_filters = tag:Role=*__: filter only the instance with _Role_ tag. More information further and in the comment inside the file.
* __boto_profile = profilename__: name of the boto profile defined in ~/.aws folder
* __cache_max_age = 0__: to force to clean cache every run.

Before launch the ansible-playbook to configure the environment is better to dry run ansible to see what is going to happen:
```
chmod +x dynamicInventory/ec2.py
ansible-playbook -i dynamicInventory/ec2.py --list-hosts site.yml
```
it will print the dry-run groups and tasks. Analyze it before run ansible-playbook.

In an AWS EC2 environment it's useful to filter and group the EC2 instances.
A good way to filter the instances is to use tags, _Role_ for example like in this repository.
The instances that have to be a webserver should have _Role=webserver_. In the _ec2.ini__settings, as describe before, we filter the instance that have a tag call _Role_.
Using _hosts: tag_Role_webserver_ as group filter in _site.yaml_ that role will be executed in the EC2 group webserver.

### Filter by environment
In addition to filter EC2 by role most of the time we need the filter also by environment, for example _webserver_ in the _development_ environment. It's possible to user multiple filters in _ec2.ini_, but there are applied with logic OR.
To solve the problem a filter is added to site.yaml in _hosts_:
* __hosts: tag_Role_value:&tag_Environment\_{{env}}__
in this way two filters liked with logic AND are applied to create group: tag Role and tag Environment. In addition an _env_ variable is added to generalized the case.
Launch the dry-run version:
```
ansible-playbook -i dynamicInventory/ec2.py --extra-vars "env=development" --list-hosts site.yaml
```

# Classic inventory
The classic version for inventory is in hosts-classic. In the file there are specified group of host
with IP, username and ssh location file. Classic ssh config file, located in ~/.ssh/config is used.
To provision the environment use:
```
ansible-playbook -i hosts-classic site.yaml
```

# Bastion Host inventory
To connect to machine trough a bastion host the hosts file (host-bastionhost) is very similar to the classic type, the difference is in the private network IP. In addition a custom __ansible.cfg__ and __ssh.cfg__ are used.
In the __ansible.cfg__ is specified to use the __ssh.cfg__ config file instead the default one. In __ssh.cfg__ is defined the bastion host to use. Then is defined a group of private IP and the proxy command to use to connect with. Bastion host and private newtwork instance have different key pair; no -A option is needed with ProxyCommand.
To provision the environment use:
```
ansible-playbook -i hosts-bastionhost site.yaml
```

# Inventory
