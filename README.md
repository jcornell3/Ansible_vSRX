# Ansible_vSRX
Proof of Concept Demonstration of Ansible deployment to a Juniper vSRX VM using NetCONF/SSH.  The example configuration includes:

Adding a new address book entry for newhost with ip x.x.x.x/32
Adding a new application with name newservice port tcp/1234
Add new policy from  any on VLAN1 (untrusted) to VLAN2 (trusted) for newservice on newhost.

## Environment
If you already have a vSRX VM working, then you don't have to use the following setup.  You'll just need to update the Ansible inventory with the appropriate attributes of your VM.

The vSRX VM (actually a Juniper Firefly Perimeter (rebranded to SRX) version 12.1X47-D15.4) is provisioned through the instructions available at [Barnesry's Junos_RPM project](https://synackattack.wordpress.com/author/barnesry/).  Downloading a recent [vSRX](https://www.juniper.net/us/en/dm/free-vsrx-trial/) would have been preferable, but these are restricted to existing Juniper customers, partners, or those who's email address indicates a good sales opportunity.

Running in a virtualenv on a OS/X 10.13.3, I installed the following components:

[Virtualbox](https://www.virtualbox.org/wiki/Downloads)

[Vagrant](https://www.vagrantup.com/downloads.html) 

For the vSRX VM provisioning, you'll need the JunOS plugin and the vSRX VM image:

```
vagrant plugin install vagrant-host-shell vagrant-junos

vagrant box add juniper/ffp-12.1X47-D15.4-packetmode
```

Since barnesry's enviroment includes exporting data from the vSRX to Graphana, the jxmlease package was necessary for the build:

```
pip install jxmlease

```

Ansible will use the NetCONF interface to JunOS devices which requires Python PyEz package junos-eznc in addition to Ansible:

```
pip install junos-eznc

pip install ansible

```

You'll want to set the path to the python interpreter in the Ansible playbook playbook-deploy-entries.yaml.  Note that Python3 is not supported by the PyEz package (as of April 2018), so make sure your environment is set to use Python 2.7:

```
# This SRX task configures a host, service and allows access to the host running the service
- name: Generate and Deploy Service Access Configuration to vsrx gateway
  hosts: vsrx
  connection: local
  gather_facts: no
  roles:
    - Juniper.junos
  vars:
    ansible_python_interpreter: "[path-to-python-interpreter]/python"

```

This can be verified using the 'ansible --version' command:

```
ansible --version
ansible 2.5.0
  config file = None
  configured module search path = [u'/Users/john/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /Users/john/vsrx/.env/lib/python2.7/site-packages/ansible
  executable location = /Users/john/vsrx/.env/bin/ansible
  python version = 2.7.10 (default, Jul 15 2017, 17:16:57) [GCC 4.2.1 Compatible Apple LLVM 9.0.0 (clang-900.0.31)]
```

Once you've completed this setup, you can just perform a 'vagrant up' to provision the environment

We'll boot the vSRX using Vagrant, and configure it using Ansible. RPM configuration will already be present (and working!) if you have internet access. You can jump on the firewall using the following command.

```
vagrant ssh vsrx
```

## Ansible Playbook Deployment

