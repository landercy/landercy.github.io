# VMWARE FUSION

## SERIAL NUMBER FOR VMWARE FUSION PRO

```
YF390-0HF8P-M81RQ-2DXQE-M2UT6
ZF71R-DMX85-08DQY-8YMNC-PPHV8
ZF3R0-FHED2-M80TY-8QYGC-NPKYF
```

## CUSTOMIZE IP RANGE FOR HOST-ONLY NETWORK

```sh
sudo vim /Library/Preferences/VMware\ Fusion/networking
```

```
VERSION=1,0
answer VNET_1_DHCP yes
answer VNET_1_DHCP_CFG_HASH 734765D0A1B8B1E10803A87231A11C1A2E5B76D1
answer VNET_1_HOSTONLY_NETMASK 255.255.255.0
answer VNET_1_HOSTONLY_SUBNET 172.16.10.0
answer VNET_1_VIRTUAL_ADAPTER yes
answer VNET_8_DHCP yes
answer VNET_8_DHCP_CFG_HASH E181887256F9B2817A7B689A27D375F8ABE54AA2
answer VNET_8_HOSTONLY_NETMASK 255.255.255.0
answer VNET_8_HOSTONLY_SUBNET 172.16.20.0
answer VNET_8_NAT yes
answer VNET_8_VIRTUAL_ADAPTER yes
```

## MOUNT SHARED FOLDER MANUALLY

```sh
yum install -y open-vm-tools
# list shared folders
vmware-hgfsclient
# mount all shared folders
vmhgfs-fuse .host:/ /mnt/hgfs
# mount specific shared folder
mkdir ~/shared
vmhgfs-fuse .host:/Downloads  ~/shared
```

## FIX NAT NETWORK

```sh
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop
sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start
```