# vagrant-ansible-role-tester
A simple vagrant framework to test ansible roles

# Requirements
You need to have some vagrant plugin installed:
```
inifile (3.0.0)
vagrant-hosts (2.8.0)
vagrant-proxyconf (1.5.2)
```

# Usage
1. Install the roles you want to test in the roles/ path

```
ansible-galaxy install -p roles/ gchiesa.shipyard
```

2. Describe the topology by editing the vagrant.yml file with the number of nodes,
groups, and variables. For instance a sample tolopogy to create a cluster of 4 nodes with CentOS/7 and install a fully functional swarm cluster with shipyard UI would be:

```
---
# TOPOLOGY
# here you can specify the topology that you want to create with vagrant in
# order to test your roles
nodes:
  - name: test01
    hostname: test01.local
    box: centos/7
    groups:
      - consul_cluster
      - swarm_managers
      - swarm_agents
  - name: test02
    hostname: test02.local
    box: centos/7
    groups:
      - consul_cluster
      - swarm_managers
      - swarm_agents
  - name: test03
    hostname: test03.local
    box: centos/7
    groups:
      - consul_cluster
      - swarm_agents
  - name: test04
    hostname: test04.local
    box: centos/7
    groups:
      - swarm_agents
# Here you can specify the variable common to all roles
common:
  ansible_host: localhost
  ansible_become: true
  ansible_ssh_user: vagrant
  ansible_ssh_private_key_file: ~/.vagrant.d/insecure_private_key
  proxy:
# Here you can specify the group variables
group_vars:
  consul_cluster:
    consul_bootstrap_hostname: test01.local

```

3. Update the information in the default.yml (with the role/roles to test) against
the hosts (all of you can use the groups specified in the vagrant.yml topology).
For instance, a default.yaml to install the shipyard cluster is the following:

```
---
- name: Testing role
  hosts: all
  any_errors_fatal: true
  pre_tasks:
    - shell: ifup eth1
      when: ansible_distribution == 'CentOS'
  roles:
    - 'gchiesa.shipyard'
```

4. Use ```vagrant up``` to startup the machines and provision them with the roles to test.
> The inventory is automatically created by the Vagrantfile based on the vagrant.yml
>topology under ```ansible/hosts```
