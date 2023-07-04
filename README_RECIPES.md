
# Build Recipes

Create specific variables files for groups of devices.
Then include in the playbook using the variable `libremesh_community_recipe`.

[[_TOC_]]

------------

### 1. Create a community recipe:
Create a configuration file inside the path defined by versions of LibreMesh and OpenWrt, and by the community name.

```
mkdir community/libremesh_master/openwrt_22.03.5/new-community/all_devices.yml
```

```
# community/libremesh_master/openwrt_22.03.5/new-community/all_devices.yml
# List of devices to build with specified profile

libremesh_devices_packages:
  - profile-valsamoggia.ninux.org-vs-ninux-generic
  - batctl-default

libremesh_devices:
  - openwrt_target: ath79
    openwrt_subtarget: generic
    openwrt_devices:
      - name: ubnt_nanostation-m-xw
      - name: ubnt_nanostation-loco-m-xw
      - name: tplink_archer-d50-v1
      - name: tplink_cpe510-v1
      - name: tplink_cpe510-v3

  - openwrt_target: ramips
    openwrt_subtarget: mt76x8
    openwrt_devices:
      - name: tplink_tl-mr6400-v4

  - openwrt_target: ramips
    openwrt_subtarget: mt76x8
    openwrt_devices:
      - name: tplink_tl-mr6400-v4

  - openwrt_target: bcm63xx
    openwrt_subtarget: generic
    openwrt_devices:
      - name: d-link_dsl-274xb-f1

  - openwrt_target: bcm63xx
    openwrt_subtarget: smp
    openwrt_devices:
      - name: sagem_fast-2704n

```
### 2. Create a lime-macaddress file

Add the property `lime_mac` to a `openwrt_device` and define:
the `lime-macaddress` and an `hostname` as they will we added to the hosts inventory file

Example:
```
# community/libremesh_master/openwrt_22.03.5/new-community/all_devices.yml
# List of devices to build with specified profile

libremesh_devices_packages:
  - profile-valsamoggia.ninux.org-vs-ninux-generic
  - batctl-default

libremesh_devices:
  - openwrt_target: ath79
    openwrt_subtarget: generic
    openwrt_devices:
      - name: ubnt_nanostation-m-xw
        lime_mac:
          - lime_mac: lime-b4fbe4598263
            hostname: mantide

```



### 3. Add specific configurations and packages in the various levels
If you need to customize some aspects of the recipe you can:
1. include or remove `packages` and `configs` referencing:
  - the whole file/recipe
  - a specific target_subtarget
  - a specific device
2. extend the loop of the devices including for specific device referenced by macaddress trough the `lime-macaddress` files

For a full list of configurations see also this file [./community/libremesh_master/openwrt_22.03.5/new-community/libremesh.defaults.yml](./community/libremesh_master/openwrt_22.03.5/new-community/libremesh.defaults.yml)

```
mkdir community/libremesh_master/openwrt_22.03.5/new-community/devices_group0.yml
```

```
# community/libremesh_master/openwrt_22.03.5/new-community/devices_group0.yml
# List of devices to build with specified profile

# Customize `versions_dist`, `version_number `and `extra_image_name` if you need to:
# version_dist: LibreMesh
# version_number: master
extra_image_name: 'group0'

# List of packages that will be used by OpenWrt ImageBuilder
# and that need to be compiled with OpenWrt Sdk 
# By default they are added from `./files/packages`
# libremesh_local_packages:
  # - custom_local_package

libremesh_devices_packages:
  - profile-valsamoggia.ninux.org-vs-ninux-generic
  - batctl-default
  # - custom_local_packages

libremesh_devices:
  - openwrt_target: ath79
    openwrt_subtarget: generic
    openwrt_devices:

# build both the `regular version` and `a firmware image for a specific device` selected by macaddress. 
# Means you have generated a specific lime-macaddress file before, using the playbook.`manage_lime_mac.yml`. 
      - name: ubnt_nanostation-m-xw
        lime_mac:
          - lime_mac: lime-b4fbe4598263
            hostname: mantide

      - name: ubnt_nanostation-loco-m-xw
        lime_mac:
          - lime_mac: lime-802aa8b71afc
            hostname: formica

# Customized packages and for this device, in this case is a test of compatibility for libremesh
# with the same packages listed at https://github.com/freifunk-gluon/gluon/blob/master/targets/ath79-generic
# if this pass the test they could be added in https://gitlab.com/a-gave/libremesh-ansible-collection/-/blob/master/target/libremesh_master/openwrt_22.03.5/ath79_generic.yml

      - name: tplink_archer-d50-v1
        packages:
        	kmod-ath10k-smallbuffers
          -kmod-ath10k-ct
          -kmod-ath10k-ct-smallbuffers
          ath10k-firmware-qca988x
          -ath10k-firmware-qca988x-ct

      - name: tplink_cpe510-v1
      - name: tplink_cpe510-v3

  - openwrt_target: ramips
    openwrt_subtarget: mt76x8
    openwrt_devices:
      - name: tplink_tl-mr6400-v4

  - openwrt_target: bcm63xx
    openwrt_subtarget: generic
    packages:
      - -procd-ujail
    openwrt_devices:
      - name: d-link_dsl-274xb-f1

  - openwrt_target: bcm63xx
    openwrt_subtarget: smp
    packages:
      - -procd-ujail
    openwrt_devices:
      - name: sagem_fast-2704n

```

### 3. Configure the build playbook with selected variables file

```
- name: Build LibreMesh {{ libremesh_version }}
  hosts: localhost
  gather_facts: no
  vars:
    libremesh_version: master
    openwrt_version: 22.03.5
    libremesh_community: new-community
    libremesh_community_recipe: devices_group0 
  roles: 
    - libremesh.libremesh.openwrt_imagebuilder_docker     
```

The output of the firmware images will be by default at:

./openwrt_build/libremesh_master/openwrt_imagebuilder_docker_22.03.5/images/new-community/devices_group0/targets/{openwrt_target}/{openwrt_subtarget}


The output of `lime-macaddress` related firmware images will be by default at:

./openwrt_build/libremesh_master/openwrt_imagebuilder_docker_22.03.5/images/new-community/devices_group0/lime-mac/<lime_macaddress>/