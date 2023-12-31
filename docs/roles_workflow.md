# Roles Workflow

[[_TOC_]]

## Build Workflow
These roles are defined in `libremesh-ansible-collection`, their tasks are to build a firmware image:
- openwrt_buildroot (based on https://gitlab.com/a-gave/ansible_openwrt_buildroot.git)
- openwrt_imagebuilder (calls openwrt_sdk if the sources of local available packages are provided)
- openwrt_imagebuilder_docker (calls openwrt_sdk_docker if the sources of local available packages are provided)

The use of these roles involve performing this operations listed:

### Build playbook setup

#### 1 - Setup a playbook defining mandatory variables
- libremesh_version (es. `2020.3`)
- openwrt_version (es. `19.07.10`)
- libremesh_community (es. `libremesh`)
- libremesh_community_recipe (es. `stable`)

#### 2 - Define a list of devices for which to build a firmware image
In the playbook itself or in `libremesh_community_recipe` at:     
./community/libremesh_{libremesh_version}/openwrt_{openwrt_version}/{libremesh_community}/{libremesh_community_recipe}.yml`

#### 3 - Launch playbook
```
ansible-playbook build_libremesh.yml
```

This playbook will execute the following operations:

### Build role workflow

#### 1 - Include variables files based on mandatory variables
Expand the variables list based on selected `mandatory variables` looking in this files:

***define default packages and configs to build libremesh for selected openwrt version***    
<libremesh-ansible-collection_installation-path>/target/libremesh_{libremesh_version}/openwrt_{openwrt_version}/main.yml
      

***define community main variables***         
./community/libremesh_{libremesh_version}/openwrt_{openwrt_version}/{libremesh_community}/main.yml      

***define variables specific to the build: libremesh_devices - the list of devices, libremesh_packages - the list of packages common to every device specified***                 
./community/libremesh_{libremesh_version}/openwrt_{openwrt_version}/{libremesh_community}/{libremesh_community_recipe}.yml       


#### 2 - Loop targets
For each openwrt target_subtarget:

##### 2.1 - Include target_subtarget related variables
***This file define a list of supported devices, and define packages and configs to keep compatibility between the default of libremesh (babeld+batman-adv), based on selected {libremesh_version} and the selected {openwrt_version}***                             
<libremesh-ansible-collection_installation-path>/target/libremesh_{libremesh_version}/openwrt_{openwrt_version}/{openwrt_target}_{openwrt_subtarget}.yml        

##### 2.2 - Print debug 
Print a debug file in /tmp/log/ with the summary of all information collected so far

##### 2.3 - Run the selected role preparing it's environment and installing it's components
- if `openwrt_buildroot` is chosen:
    - install the requirements
    - clone specified openwrt_buildroot version in the folder {openwrt_dir}/libremesh_{libremesh_version}openwrt_buildroot_{openwrt_version}
    - add libremesh feeds
- if `openwrt_imagebuilder` is chosen:
    - install the requirements
    - clone specified openwrt_imagebuilder version in the folder {openwrt_dir}/libremesh_{libremesh_version}openwrt_imagebuilder_{openwrt_version}
    - add libremesh feeds

#### 3 - Configure libremesh
Configure libremesh for selected:
- libremesh_version/openwrt_version 
- target_subtarget

With packages and configurations selected in the included variables file:
- libremesh_version_packages (es. `-dnsmasq -ppp -odhcp_ipv6only`)
- libremesh_target_subtarget_packages (es `-procd-ujail` for bcm63xx)
- libremesh_community_recipe_packages (es. `profile-libremesh-suggested-packages` [0] `luci`)
- libremesh_community_recipe_target_subtarget_packages (device packages related to target_subtarget)

[0] This select the recommended packages:
- based on the documentation on the site of libremesh at https://libremesh.org/development
- through the package defined in the network profile of libremesh at https://github.com/libremesh/network-profiles/tree/master/libremesh/suggested-packages
- more information at https://libremesh.org/development-network_profiles.html

#### 4 - Loop devices
For each openwrt target_subtarget_device:

##### 4.1 - Configure single device
Configure libremesh for specified device with packages and configurations selected in the included variables file:
- libremesh_target_subtarget_device_packages (device specific packages to work with libremesh default)
- libremesh_community_recipe_target_subtarget_device_packages (device specific packages based on choosen profile)

##### 4.2 - Print debug of single device: configs and packages
Add information to the debug file in /tmp/log/ with selected configs for the device

##### 4.3 - Build LibreMesh
Build for the selected device
