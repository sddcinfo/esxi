# esxi
The goal was to be able to automatically install esxi on a UEFI client using TFTP and autoinstall the image.
# dnsmasq
Required dnsmasq settings to boot UEFI
### settings for UEFI boot
```
dhcp-match=set:efi-x86_64,option:client-arch,7
dhcp-boot=tag:efi-x86_64,mboot.efi
### TFTP Server setup
enable-tftp
tftp-root=/var/lib/tftpboot
```
## tftp-root
```
mkdir -p /var/lib/tftpboot
mkdir /tmp/iso
mount -o loop esxi-6.7.0.u1-10302608.x86_64.iso /iso/
cp -rf /tmp/iso/ /var/lib/tftpboot/esxi67u1
umount /tmp/iso

sed -i 's/\///g' /var/lib/tftpboot/esxi67u1/boot.cfg
cp /var/lib/tftpboot/esxi67u1/efi/boot/bootx64.efi /var/lib/tftpboot/esxi67u1/mboot.efi

cp /var/lib/tftpboot/esxi67u1/boot.cfg /var/lib/tftpboot/boot.cfg
cp /var/lib/tftpboot/esxi67u1/mboot.efi /var/lib/tftpboot/mboot.efi
```
You need to do one change in the boot.cfg file copied to the tftpboot root folder, namely specify the prefix option to link it to the folder where your image files reside:

```
$ head /var/lib/tftpboot/boot.cfg
bootstate=0
title=Loading ESXi installer
timeout=5
prefix=esxi67u1
kernel=b.b00
kernelopt=ks=http://10.10.1.1/ks.cfg

```
## ks.cfg
```
#Accept VMware License agreement
accepteula
# Set the root password
rootpw VMware1!
# Following clear all partitions on the local disks
clearpart --alldrives --overwritevmfs
# Install ESXi on the first disk (Local first, then remote then USB)
install --firstdisk --overwritevmfs
# Set the network
network --bootproto=dhcp --device=vmnic0
# reboot the host after installation is completed
reboot
```
# bootdev
It's important to ensure you have the boot order set as HDD first, then PXE.
See https://github.com/sddcinfo/supermicro/blob/master/bios.txt
```bash
ipmitool -H $HOST -U $USER -P $PASS chassis bootdev pxe options=efiboot
```
This will force the server to boot PXE on the next boot only.
