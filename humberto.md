## Prerequisites

Run everything as root

Add `clear` user to `kvm` group

```shell script
sudo usermod -a -G kvm clear
```

Chmod /dev/kvm to workaround [bug](https://bugzilla.redhat.com/show_bug.cgi?id=1479558)

```shell script
sudo chmod 666 /dev/kvm
sudo sed -i s/0660/0666/g /lib/udev/rules.d/80-kvm.rules
```

Install sysadmin-basic bundle for `tput` binary

```shell script
sudo swupd bundle-add sysadmin-basic # tput
sudo swupd bundle-add  dhcp-server vim # for dnsmasq
sudo swupd bundle-add network-basic # bridge-utils
```

Stop NetworkManager if it is running

```shell script
systemctl stop NetworkManager.service
systemctl mask NetworkManager.service
```

Edit network scripts. Replace eno2 and eno1 with your active interface

```shell script
#Determine your active interface
ip a | grep UP

#Change eno1 to your active interface
sudo vi configs/host/clear/50-local-external-nic.network

#Update MACAddress for the active interface in 50-external-bridge.network
sudo vi /etc/systemd/network/50-external-bridge.network
[Link]
MACAddress=

#Remove 50-dhcp.network
```

Copy network scripts

```shell script
sudo mkdir -p /etc/systemd/network/
sudo cp -R configs/host/clear/* /etc/systemd/network/
sudo systemctl enable systemd-resolved
sudo systemctl daemon-reload
sudo systemctl restart systemd-networkd
ip a   # ponte should be UP and have ip assigned, the interface should give up its ip
```

Make shell scripts executable

```shell script
sudo chmod +x scripts/tests/*.sh
sudo chmod +x percoso/configs/net/*
```

Generate key for your host's root user
```shell script
ssh-keygen -t rsa
ls /root/.ssh/  # should output id_rsa  id_rsa.pub
```

## Usage
If running outside of the Intel Proxy then set the environment variable EXTERNAL_HOST_NETWORK. If inside Intel Proxy then unset environment variable EXTERNAL_HOST_NETWORK

```shell script
cd percoso
EXTERNAL_HOST_NETWORK=1
./scripts/tests/setup_k8s-cluster.sh
```

I am running inside the Intel Proxy.

To reduce the memory for VMs, because of lack of resources on host
```shell script
sudo vi lib/bash-funcs

# edit the following RAM and CPU count
local KVM_CMD="qemu-system-x86_64                                                         \
            -enable-kvm -m 4G -smp 2 -cpu host -machine accel=kvm,usb=off -nodefaults        \
            -device virtio-balloon-pci,id=balloon0 -overcommit mem-lock=off -msg timestamp=on \
```

##Cleanup
Do the following for a fresh start before running the script from scratch
```shell script
sudo rm -rf /tmp/
sudo rm -rf /scripts/tests/vms
```
