# F5 and Ansible Demo Deployment

* Basic series of onboarding steps to bootstrap a BIG-IQ system [f5devcentral/ansible-role-bigiq_onboard](https://github.com/f5devcentral/ansible-role-bigiq_onboard)

## Ansible Installation

### Ubuntu

```shell
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install software-properties-common
$ sudo apt-add-repository --yes --update ppa:ansible/ansible
$ sudo apt install ansible
```
### macOS
```shell
$ brew install ansible
```
or
```shell
$ pip install ansible
$ pip install jmespath
```

* in my previous installation on my _macOS_, _Ansible_ was crashing, unless I installed *Python v3.7* and pointed the interpreter in the configuration to this version `./ansible.cfg`:
  ```
  [defaults]
  interpreter_python = /usr/local/opt/python@3.7/libexec/bin/python
  ```

  > This is not the case anymore

## Config Files
~/.ansible.cfg:
```
[defaults]
connection = smart
timeout = 60
host_key_checking = False
interpreter_python = auto_silent
```

./hosts:
```
[all:vars]
ansible_user=centos
ansible_ssh_pass=
ansible_ssh_private_key_file=/home/centos/.ssh/aws-private.pem

[lb]
f5 ansible_host=10.1.20.7 ansible_user=admin private_ip=10.1.1.7 destination_ip=10.1.20.100 ansible_ssh_pass=

[control]
ansible ansible_host=10.1.1.4 ansible_user=centos

[webservers]
host1 ansible_host=10.1.20.5 ansible_user=centos private_ip=10.1.1.5
host2 ansible_host=10.1.20.6 ansible_user=centos private_ip=10.1.1.6
```

## Ansible notes

* use `ansible-playbok -i <hosts> playbook` option to specify the hosts file or use _inventory_ option in the `~/.ansible.cfg` file

## Demo Notes
- Ansible part is based on the *F5 Agility 2020 - 🦅 Ansible Lab 101*, F5ers can use the UDF
- for local deployment use F5-CLI/DO deployment from my [f5-demo-lab](https://github.com/erkac/f5-demo-lab) in order to onboard and license the *BIG-IP*

## Playbook examples

### ./playbooks/1-bigip-facts.yml
```shell
$ ansible-playbook -i hosts ./playbooks/1-bigip-facts.yml
$ # OR
$ ansible-playbook -i hosts ./playbooks/1-bigip-facts.yml --skip-tags=debug
```
- skips tasks with specific tag _debug_
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
- iRule1 and iRule2 has to be in the current folder, even the ansible would find those in subfolders, but then creating that iRule on F5 cause issues if the name contains some subfolders

### ./playbooks/7-bigip-config.yml
- **configuration** has to be **saved manually**

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
- add the _rescue_ stanza. The tasks under the rescue stanza will be identical to ./playbooks/9-bigip-delete-configuration.yml. The bigip_pool_member task does not need to re-entered since by deleting the nodes and pool will remove all configuration. If any task within the block fails, the _rescue_ stanza will execute in order. The VIP, pool, and nodes will be removed gracefully
- add the always to save the running configuration

## Ansible F5 AS3 Exercises ./playbooks/11-as3.yml
- [FAS Ansible Workshop 101 > Exercise 3.0 - Introduction to AS3](https://clouddocs.f5.com/training/fas-ansible-workshop-101/3.0-as3-intro.html)

- AS3 requires a JSON template to be handed as an API call to F5 BIG-IP.
- `tenant_base.j2` is a standard template that F5 Networks will provide to their customers. The important parts to understand are:
  - `"WorkshopExample": {` - this is the name of our Tenant. The AS3 will create a tenant for this particular WebApp. A WebApp in this case is a virtual server that load balances between our two web servers.
  - `"class": "Tenant",` - this indicates that WorkshopExample is a Tenant.
  - `as3_app_body` - this is a variable that will point to the second _jinja2_ template which is the actual WebApp.

- This template `as3_template.j2` is a JSON representation of the Web Application. The important parts to note are:
  - There is a virtual server named _serviceMain_.
    - The template can use variables just like tasks do in previous exercises. In this case the virtual IP address is the _private_ip_ from our inventory.
  - There is a _Pool_ named _app_pool_
    - The _jinja2_ template can use a loop to grab all the pool members (which points to our web servers group that will be elaborated on below).
- In Summary the `tenant_base.j2` and `as3_template.j2` create one single JSON payload that represents a Web Application. We will build a Playbook that will send this JSON payload to a F5 BIG-IP.