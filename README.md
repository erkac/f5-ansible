# Ansible for F5

## Config Files
~/.ansible.cfg:
```
[defaults]
connection = smart
timeout = 60
inventory = /home/centos/networking-workshop/lab_inventory/hosts
host_key_checking = False
private_key_file = /home/centos/.ssh/aws-private.pem
```

lab_inventory/hosts:
```
[all:vars]
ansible_user=centos
ansible_ssh_pass=f5ansible
ansible_ssh_private_key_file=/home/centos/.ssh/aws-private.pem

[lb]
f5 ansible_host=10.1.20.7 ansible_user=admin private_ip=10.1.1.7 destination_ip=10.1.20.100 ansible_ssh_pass=f5ansible

[control]
ansible ansible_host=10.1.1.4 ansible_user=centos

[webservers]
host1 ansible_host=10.1.20.5 ansible_user=centos private_ip=10.1.1.5
host2 ansible_host=10.1.20.6 ansible_user=centos private_ip=10.1.1.6
```

## Playbook examples

### ./playbooks/1-bigip-facts.yml
```
$ ansible-playbook bigip-facts.yml
$ ansible-playbook bigip-facts.yml --skip-tags=debug
```
- skips tasks with spacific tag _debug_
- in this case output from _COLLECT BIG-IP FACTS_ is stored in _device_facts_
- displayed by the _DISPLAY COMPLETE BIG-IP SYSTEM INFORMATION_ task using var: device_facts

### ./playbooks/2-bigip-node.yml
- A _loop_ will repeat a task on a list provided to the task. In this case it will loop twice, once for each of the two web servers.

### ./playbooks/3-bigip-pool.yml
- The _monitor_type: "and_list"_ ensures that all monitors are checked

### ./playbooks/4-bigip-pool-members.yml
- The _state: "present"_ parameter tells the module we want this to be added rather than deleted.

### ./playbooks/4a-display-pool-members.yml.yml
- _vars:_ in the module is defining a variable query_string to be used within the module itself
- _query_String_ will have the name of all members from pool name ‘http_pool’. query_string is defined to make it easier to read the entire json string

### ./playbooks/5-bigip-virtual-server.yml
🥳

### ./playbooks/6-bigip-irule.yml
- irule1 and irule2 has to be in the current folder, even the ansible whould find those in subfolders, but then creating that iRule on F5 cause issues if the name contains some subfolders

### ./playbooks/7-bigip-config.yml
- configuration has to be saved manualy

### ./playbooks/8-disable-pool-member.yml
- Once you set the provider you can re-use this key in future tasks instead of giving the server/user/password/server_port and validate_certs info to each task.
- Retrieve Facts from BIG-IP for the subset ltm-pools
- Display the pool information to the terminal window
- Store the pool name as a fact
- Display members belonging to the pool
- Prompt the user to enter a Host:Port to disable a particular member or ‘all’ to disable all members
- Read the prompt information and disable all members or a single member based on the input from the user

### ./playbooks/9-bigip-delete-configuration.yml
- The _state: absent_ will remove the configuration from the F5 BIG-IP load balancer

### ./playbooks/10-bigip-error-handling.yml
- add the _block_ stanza and the first task
- add the _rescue_ stanza. The tasks under the rescue stanza will be identical to ./playbooks/9-bigip-delete-configuration.yml. The bigip_pool_member task does not need to re-enterered since by deleting the nodes and pool will remove all configuration. If any task within the block fails, the _rescue_ stanza will execute in order. The VIP, pool, and nodes will be removed gracefully
- add the always to save the running configuration


## Ansible F5 AS3 Exercises
- To be continued...