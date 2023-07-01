# LibreMesh Ansible playbooks

This project defines some ansible playbook templates to use [Libremesh Ansible collection](https://gitlab.com/a-gave/libremesh-ansible-collection.git)

They are intended to do common network administration tasks 
useful for a tech team of a community-network based on LibreMesh.

[[_TOC_]]


Requirements
------------

Install Ansible requirements:

    apt install ansible python3-pip
    pip3 install jinja2-ansible-filters

To use the ansible commands `ansible-playbook`, `ansible-galaxy`
Add the `~/.local/bin` path to your .bashrc or .bash_profile

    echo "export PATH=$PATH:$HOME/.local/bin" >> ~/.bashrc
    source ~/.bashrc

Clone this project
```
    git clone https://gitlab.com/a-gave/libremesh-ansible-playbooks.git
    cd libremesh-ansible-playbooks
```

Dependencies
------------
Install roles and collections on which this collection depends

    ansible-galaxy install -r requirements.yml


Get started
------------

### 1. Create an inventory file
https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.html

    cp hosts.example hosts

Adjust the inventory file defining:
  - hostname: here is `buildhost`
  - ip_address: here is `12.34.56.78`
  - user for ssh connection: here is `builder` 

```
# hosts.example
buildhost:
  ansible_host: 12.34.56.78
  ansible_user: builder
  ansible_become_pass: "{{ builder_become_pass }}"
  ansible_become_user: root
  ansible_become_method: su
  ansible_become_flags:
```

### 2. Setup a system to retrieve passwords
------------

#### 2.1 using ansible-vault
https://docs.ansible.com/ansible/latest/cli/ansible-vault.html

Create a vault password

    echo 'CHANGEME' > .vault  

Create a vault protected file,
For simplicity create the variable in `group_vars/all.yml` to be sure to include it whatever host will run the playbook

    ansible-vault create group_vars/all.yml --vault-password-file=".vault"
    
Add `buildhost` root password

    builder_become_pass: ROOT_PASSWORD

#### 2.2 using community.general.passwordstore lookup
https://docs.ansible.com/ansible/latest/collections/community/general/passwordstore_lookup.html

For simplicity create the variable in `group_vars/all.yml` to be sure to include it whatever host will run the playbook

    mkdir group_vars/

Add the path to find the key in your passwordstore, in this example is `buildhost/user/root`

    cat << EOF >> group_vars/all.yml
    
    builder_become_pass: "{{ lookup('passwordstore', 'buildhost/user/root', errors='strict') | default(omit) }}"
    EOF

### 3. Setup the ansible local configuration file
https://docs.ansible.com/ansible/latest/reference_appendices/config.html

Copy the default ansible configuration file

    cp ansible.cfg.example ansible.cfg

Adjust ansible local configuration file uncommenting `vault_password_file` if you intend to use ansible-vault
```
# ansible.cfg.example
[passwordstore_lookup]
lock = readwrite
locktimeout = 45000s

[defaults]
inventory = ./hosts
#callbacks_enabled = profile_tasks
interpreter_python = /usr/bin/python3
remote_user = root
# vault_password_file = ./.vault

[ssh_connection]
scp_if_ssh = True
```

### 4. Setup the example playbook
Select devices of which to build a libremesh firmware image, in the playbook file `build_libremesh.yml`. In this example the selected device is `ubnt_nanostation-m-xw`.

```
---
- name: Build LibreMesh {{ libremesh_version }}
  hosts: localhost
  gather_facts: no
  vars:
    libremesh_version: master
    openwrt_version: 22.03.5
    libremesh_community: libremesh
    libremesh_community_recipe: stable
    openwrt_targets:
      - openwrt_target: ath79
        openwrt_subtarget: generic
        openwrt_devices: 
          - name: ubnt_nanostation-m-xw    
  roles: 
    - libremesh.libremesh.openwrt_imagebuilder_docker   
```

This select a default recipe located at `community/libremesh_master/openwrt_22.03.5/libremesh/stable.yml` that include the default set of packages for libremesh using the network profiles https://github.com/libremesh/network-profiles.git

Read also [README_RECIPES.md](./README_RECIPES.md) for an explanation of the configurations files.


### 5. Build LibreMesh

    ansible-playbook build_libremesh.yml

Read also [README_ROLES.md](./README_ROLES.md) for an explanation of the build workflow.


Overview
------------

### Build firmware images
#### Create a community recipe
The variables file by default should be included in a path defined by the version of libremesh and the version of openwrt that should be used, this eases and rends explicit the matching of configurations that should be applied, and is suitable e.g. for a small tech team that needs to build firmware images for different devices with configurations that may vary depending on LibreMesh or OpenWrt development.

Use the community recipe to define:
- list of target_subtarget/devices of which build a firmware image 
- packages that are common to multiple devices, and to multiple target_subtarget 
- packages that depends on target_subtarget or device
- configuration files (`lime-macaddress`) and/or packages for specific devices
- environment variables about where to execute the build, where binaries should be produced

#### Generate device specific configurations 
Generare a device specific configuration file based on macaddress using the playbook `manage_lime_mac.yml`

### Monitoring
- Setup a monitoring system on a server
    - prometheus
    - alertmanager
    - blackbox_exporter
    - grafana
- Setup a monitoring system with a vpn
- Add LibreMesh devices to a monitoring system


