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
ansible-galaxy install -p roles/ gchiesa.consul
```

2. Describe the topology by editing the vagrant.yml file with the number of nodes,
groups, and variables

3. Update the information in the default.yml (with the role/roles to test) against
the hosts (all of you can use the groups specified in the vagrant.yml topology)

4. Use vagrant up to startup the machines and provision them with the roles to test.
The inventory is automatically created by the Vagrantfile based on the vagrant.yml
topology
