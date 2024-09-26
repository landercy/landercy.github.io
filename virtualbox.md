# VIRTUALBOX

## NETWORK

- File > Host Network Manager > Create
  - vboxnet0
     - Configure Adapter Manually
        - IPv4 Address: 192.168.56.1
        - IPv4 Network Mask: 255.255.255.0
     - DHCP Server: Disable
- VM Settings > Newwork
  - Adapter 1: NAT
  - Adapter 2: Host-only Adapter > vboxnet0

## RUM VM IN BACKGROUND

```sh
# list
VBoxManage list vms
VBoxManage list runningvms

# state
vboxmanage list -l vms|grep State

# start
VBoxManage startvm centos --type headless

# poweroff
VBoxManage controlvm centos poweroff

# pause
VBoxManage controlvm centos pause

# resume
VBoxManage controlvm centos resume
```

## VBOX ADDTIONS

http://download.virtualbox.org/virtualbox/5.2.0_RC1/VBoxGuestAdditions_5.2.0_RC1.iso

```sh
yum install -y epel-release 
reboot
yum install -y kernel-devel-$(uname -r) gcc make bzip2
```

## MOUNT SHARED FOLDER MANUALLY

```sh
mount -t vboxsf centos sharedfolder
```