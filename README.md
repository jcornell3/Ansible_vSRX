# Ansible_vSRX
Proof of Concept Demonstration of Ansible deployment to a Juniper vSRX VM using NetCONF/SSH.  The example configuration includes:

Adding a new address book entry for newhost with ip x.x.x.x/32. I've used a local address of 192.168.1.15/32 in this example.

Adding a new application with name newservice port tcp/1234.

Add new policy from  any on VLAN1 (untrusted) to VLAN2 (trusted) for newservice on newhost.

## Environment
If you already have a vSRX VM working, then you don't have to use the following setup.  You'll just need to add the junos-eznc package, add python 2.7 (if not already installed) and update the Ansible inventory and playbook with the appropriate attributes of your VM.

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

We'll boot the vSRX using Vagrant, and configure it using Ansible. The vSRX configuration will already be present (and working!) if you have internet access. You can jump on the firewall using the following command.

```
vagrant ssh vsrx
```

## Ansible Playbook Deployment

At this point you can run the Ansible playbook with the following command, which references the configuration file [vsrx.conf.s2](https://github.com/jcornell3/Ansible_vSRX/blob/master/provisioning/vsrx.conf.s2) to be pushed to the vSRX:

ansible-playbook -i [path to inventory]/vagrant_ansible_inventory [path to playbook]/playbook-deploy-entries.yaml

The result should look like the following:

```
 [WARNING]: Found both group and host with same name: vsrx


PLAY [Generate and Deploy Service Access Configuration to vsrx gateway] *********************************************************************

TASK [deploy service access config] *********************************************************************************************************
ok: [vsrx]

TASK [Display all variables/facts known for a host] *****************************************************************************************
ok: [vsrx] => {
    "hostvars[inventory_hostname]": {
        "ansible_check_mode": false,
        "ansible_diff_mode": false,
        "ansible_facts": {},
        "ansible_forks": 5,
        "ansible_host": "127.0.0.1",
        "ansible_inventory_sources": [
            "/Users/john/vsrx/Ansible_vSRX/provisioning/inventory/vagrant_ansible_inventory"
        ],
        "ansible_playbook_python": "/usr/local/opt/python/bin/python3.6",
        "ansible_port": 2200,
        "ansible_run_tags": [
            "all"
        ],
        "ansible_skip_tags": [],
        "ansible_ssh_private_key_file": "/Users/john/.vagrant.d/insecure_private_key",
        "ansible_user": "vagrant",
        "ansible_version": {
            "full": "2.5.0",
            "major": 2,
            "minor": 5,
            "revision": 0,
            "string": "2.5.0"
        },
        "group_names": [
            "ungrouped"
        ],
        "groups": {
            "all": [
                "ubuntu-monitoring",
                "vsrx"
            ],
            "ungrouped": [
                "ubuntu-monitoring",
                "vsrx"
            ],
            "vsrx": []
        },
        "host": {
            "loopback": {
                "ip": "192.168.0.1"
            }
        },
        "interfaces": [
            {
                "description": "to_vboxnet0_mangement",
                "ip": "192.168.56.107",
                "name": "ge-0/0/1"
            }
        ],
        "inventory_dir": "/Users/john/vsrx/Ansible_vSRX/provisioning/inventory",
        "inventory_file": "/Users/john/vsrx/Ansible_vSRX/provisioning/inventory/vagrant_ansible_inventory",
        "inventory_hostname": "vsrx",
        "inventory_hostname_short": "vsrx",
        "omit": "__omit_place_holder__e1b09e490219d2f22956ba7228a0e2088a83421e",
        "playbook_dir": "/Users/john/vsrx/Ansible_vSRX/provisioning"
    }
}
[DEPRECATION WARNING]: junos_install_config is kept for backwards compatibility but usage is discouraged. The module documentation details 
page may explain more about this rationale.. This feature will be removed in a future release. Deprecation warnings can be disabled by 
setting deprecation_warnings=False in ansible.cfg.

TASK [Deploy config to device ... please wait] **********************************************************************************************
ok: [vsrx]

TASK [Checking NETCONF connectivity] ********************************************************************************************************
ok: [vsrx]

PLAY RECAP **********************************************************************************************************************************
vsrx                       : ok=4    changed=0    unreachable=0    failed=0   

```