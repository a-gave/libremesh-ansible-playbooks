
# libremesh_playbooks


Build LibreMesh images with docker:
pros: 
- save space (?amount of space need, checkout https://github.org/openwrt/docker), 
- avoid install openwrt build-system requirements, and install only docker


# Notes: change to a ansible docker module? currently the docker operations are handled by the shell module
By default the roles openwrt_imagebuilder_docker and openwrt_sdk_docker try to stop and remove respectively all container whose image start with 'openwrt\imagebuilder' and 'openwrt\sdk', to avoid leave them running in the background if the previous play fails or is interrupted during build and so the container is not removed. if you need to disable this for somewhat reasons use:
disable_stop_all_openwrt_container_matching:




# compile libremesh only for all supported devices