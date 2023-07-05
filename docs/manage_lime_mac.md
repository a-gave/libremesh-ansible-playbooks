# Manage lime mac

[[_TOC_]]

## Setup the playbook

Edit the playbook file `manage_lime_mac.yml` selecting the chosen recipe.

```
---
- name: Manage lime mac
  hosts: localhost
  gather_facts: no
  vars:
    libremesh_version: master
    openwrt_version: 22.03.5   
    libremesh_community: new-community
    libremesh_community_recipe: example_recipe
    # skip_init_vars_lime_mac: true                      # skip this if initialization of lime-mac variables doesn't need to be executed
    # skip_generate_lime_mac: false                      # don't skip if you already had configured the hostvars/lime_mac files for each devices
    # skip_update_vpn_wg_server: false                   # don't skip if you are ready to update wireguard server
  roles: 
    - libremesh.libremesh.manage_lime_mac

```


## Setup the recipe

### Define the network options in the recipe file

For example to keep the default libremesh values add these variables in the recipe file `example_recipe`.

```
default_channel_5ghz: 48
ipv4_network: "10.13"
ipv4_netmask: "/16"
```

### Define the list of specific devices in the recipe file

Setup the recipe that define a list of `lime-macaddress` as described at [./recipes.md](./recipes.md)

Example of recipe that define specific device

```
---
default_channel_5ghz: 48
ipv4_network: "10.13"
ipv4_netmask: "/16"

libremesh_packages:
  - profile-libremesh-default
  - profile-libremesh-suggested-packages

libremesh_devices:
  - openwrt_target: ath79
    openwrt_subtarget: generic
    openwrt_devices:
      - name: ubnt_nanostation-m-xw
        lime_mac:
          - lime_mac: lime-b4fbe4598263
            hostname: mantide

      - name: ubnt_nanostation-loco-m-xw
        lime_mac:
          - lime_mac: lime-802aa8b71afc
            hostname: formica
```

### Initialize the devices as ansible hosts

This command will generate an inventory file at `inventory/libremesh_devices.yml` with hosts selected by the chosen recipe.
And a variables file for each device located at `host_vars/<lime-macaddress>.yml`.

```
ansible-playbook manage_lime_mac.yml
```

### Generate a lime-macaddress file for the device

Edit the variables file at `host_vars/<lime-macaddress>` to add configurations or modify default values.

Then run again the playbook skipping the initialization to generate the lime-macaddress files under `./lime-mac/<lime-macaddress>`

```
ansible-playbook -i inventory/ manage_lime_mac.yml -e "{ skip_init_vars_lime_mac: true, skip_generate_lime_mac: false }" 
```


## Add Wireguard keys generation

### Generate wireguard keys as hostvars

Define network related variables for the wireguard vpn in the file of the chosen recipe. 
```
# VPN options
default_vpn_wg0_listenport: 51800
vpn_wg0_network: "192.168"
vpn_wg0_netmask: "/16"
```

Re-run the playbook to generate also the wireguard keys 
```
ansible-playbook manage_lime_mac.yml
```

### Add the device as a peer in vpn server based on wireguard

Requires a host with a public ip that acts as a vpn server defined as `wg_server` in an inventory file.

```
ansible-playbook -i inventory/ manage_lime_mac.yml -e "{ skip_init_vars_lime_mac: true, skip_update_vpn_wg_server: false }" 
```