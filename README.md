
# libremesh_playbooks

Here are defined some playbook templates to do common network administration tasks 
useful for a tech team of a community-network based on LibreMesh.


## Get started

0. Initialize community variables 
  - Create community variables files with
    - list of target_subtarget/devices of which build a firmware image 
    - packages that are common to multiple devices, and to multiple target_subtarget 
    - packages that depends on target_subtarget or device
    - configuration files (lime-macaddress) and/or packages for specific devices
    - environment variables about where to execute the build, where binaries should be produced

1. Generate device specific configurations file

2. Build Libremesh

3. Monitoring
  - Setup a monitoring system on a server
    - prometheus
    - alertmanager
    - blackbox_exporter
    - grafana
  - Setup a monitoring system with a vpn
  - Add LibreMesh devices to a monitoring system

---

0. Initialize community variables
The variables file by default should be included in a path defined by the version of libremesh and the version of openwrt that should be used, this ease and rend explicit the matching of configurations that should be applied, and is suitable e.g. for a small tech team that needs to build for different devices with configurations that may vary depending on LibreMesh or OpenWrt repositories

The example playbooks 

### 0.1 Create a directory inside the libremesh_version/openwrt_version with the name of your community
```
mkdir libremesh_2023.1/openwrt_22.03.5/new_community.yml
```

If you prefer to base openwrt and/or libremesh on `branchs` instead of `tags` copy the corresponding tags folder
```
cp -r libremesh_2023.1/openwrt_22.03.5 libremesh_master/openwrt_22.03
```


### 0.2 Copy the template of a community variables folder
```
cp -r libremesh_2023.1/openwrt_22.03.5/new-community/main.defaults.yml  libremesh_2023.1/openwrt_22.03.5/new-community/main.yml
```


### 0.3 Define in the main file the system user and main openwrt_dir




