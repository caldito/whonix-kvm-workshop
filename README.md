# whonix-kvm-workshop
Repo for following the Whonix in KVM workshop

## Requirements
* Install instructions tested in Debian Bookworm 
* User account in sudo group
## Resources
* [Whonix KVM](https://www.whonix.org/wiki/KVM)
* [Whonix KVM (onion version)](http://www.dds6qkxpwdeubwucdiaord2xgbbeyds25rbsgr73tbfpqpt4a6vjwsyd.onion/wiki/KVM)
* [GPG cheat sheet](https://gock.net/blog/2020/gpg-cheat-sheet)
## Installation steps
### Installing QEMU/KVM, libvirt and virt-manager
Debian bookworm
```
sudo apt update
sudo apt install --no-install-recommends qemu-kvm qemu-system-x86 libvirt-daemon-system libvirt-clients virt-manager gir1.2-spiceclientgtk-3.0 dnsmasq-base qemu-utils iptables safe-rm xz-utils
sudo adduser "$(whoami)" libvirt
sudo adduser "$(whoami)" kvm
sudo systemctl restart libvirtd

```
Make sure default network is enabled and started. Not needed but useful for other VMs
```
sudo virsh -c qemu:///system net-autostart default
sudo virsh -c qemu:///system net-start default
```
### Adding Whonix download, verification and extraction
Download Whonix VMs, verify and extract
```
wget https://download.whonix.org/libvirt/17.2.3.7/Whonix-Xfce-17.2.3.7.Intel_AMD64.qcow2.libvirt.xz
wget https://www.whonix.org/download/libvirt/17.2.3.7/Whonix-Xfce-17.2.3.7.Intel_AMD64.qcow2.libvirt.xz.asc
wget https://www.whonix.org/keys/derivative.asc
gpg --import derivative.asc
gpg --verify-options show-notations --verify Whonix-*.libvirt.xz.asc Whonix-*.libvirt.xz

```
The file is (most likely) correct if we see in the ouput something like:
```
gpg: Good signature from "Patrick Schleizer <adrelanos@kicksecure.com>" [unknown]
gpg:                 aka "Patrick Schleizer <adrelanos@riseup.net>" [unknown]
gpg:                 aka "Patrick Schleizer <adrelanos@whonix.org>" [unknown]
```

You'll see a warning which is normal because the key hasn't been assigned enough level of trust. If you are paranoid verify that the key is correct:
```
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
```

Extract the compressed file:
```
tar -xvf Whonix*.libvirt.xz
```
### Whonix network setup
Prepare the Whonix networks with the virsh cli:
```
sudo virsh -c qemu:///system net-define Whonix_external*.xml
sudo virsh -c qemu:///system net-define Whonix_internal*.xml
sudo virsh -c qemu:///system net-autostart Whonix-External
sudo virsh -c qemu:///system net-start Whonix-External
sudo virsh -c qemu:///system net-autostart Whonix-Internal
sudo virsh -c qemu:///system net-start Whonix-Internal
```
### Whonix VM and disks setup
Define the Whonix virtual machines with the virsh cli:
```
sudo virsh -c qemu:///system define Whonix-Gateway*.xml
sudo virsh -c qemu:///system define Whonix-Workstation*.xml
```
At this point the machines should be visible in the virt-manager GUI but wouldn't work if started because the disks are not in the correct path.

Place the virtual disk in their path:
```
sudo mv Whonix-Gateway*.qcow2 /var/lib/libvirt/images/Whonix-Gateway.qcow2
sudo mv Whonix-Workstation*.qcow2 /var/lib/libvirt/images/Whonix-Workstation.qcow2
```

Now you can start the VMs in virt-manager UI.
First the Gateway and verify that tor is running in the top right corner after the system check completes.
Then the Workstation
## Troubleshooting

Fix missing dnsmasq dir when starting VMs:
```
sudo mkdir /var/lib/libvirt/dnsmasq
sudo chmod 711 /var/lib/libvirt/dnsmasq
```
Apt not updating because time related issues: [Network Time Synchronization](https://www.whonix.org/wiki/Network_Time_Synchronization)
```

```
