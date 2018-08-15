# Guide of PXEboot & Windows install on ZFS with iSCSI.
## Machine Spec
- iSCSI Target
  - NEC VersaPro (core i5, mem:4GB, nic:Intel)
  - OS: CentOS 7.5
- iSCSI Initiator
  - Thinkpad L530 (core i5, mem:4bg, nic:Realtec)

## iSCSI Target Settings
- First, When installing CentOS, it's easy to set ZFS by separating unused partitions for ZFS.

### Install ZFS
```
- \# rpm -Uvh http://archive.zfsonlinux.org/epel/zfs-release.el7.noarch.rpm
- \# yum install -y epel-release
- \# yum install -y kernel-devel zfs
- \# zpool list & lsmod | zfs
  - [no pools available] is OK.
- \# fdisk /dev/sdb
  - create zfs partition
- \# fdisk -l
  - check zfs partition
  - ex) /dev/sdb3
- \# zpool create -f zfs /winpool sdb3
  - create windows pool on zfs storage
- \# df -h
  - check zfs pool (winpool)
- \# zfs list
  - check zfs pool (winpool)
```

## Install DHCP Server
- \# yum -y install dhcp
- Add the following to dhcpd.conf
  - Subnet address, MAC address, etc should be adapted to each environment.
```
option space ipxe;
option ipxe-encap-opts code 175 = encapsulate ipxe;
option ipxe.priority code 1 = signed integer 8;
option ipxe.keep-san code 8 = unsigned integer 8;
option ipxe.skip-san-boot code 9 = unsigned integer 8;
option ipxe.syslogs code 85 = string;
option ipxe.cert code 91 = string;
option ipxe.privkey code 92 = string;
option ipxe.crosscert code 93 = string;
option ipxe.no-pxedhcp code 176 = unsigned integer 8;
option ipxe.bus-id code 177 = string;
option ipxe.san-filename code 188 = string;
option ipxe.bios-drive code 189 = unsigned integer 8;
option ipxe.username code 190 = string;
option ipxe.password code 191 = string;
option ipxe.reverse-username code 192 = string;
option ipxe.reverse-password code 193 = string;
option ipxe.version code 235 = string;
option iscsi-initiator-iqn code 203 = string;
# Feature indicators
option ipxe.pxeext code 16 = unsigned integer 8;
option ipxe.iscsi code 17 = unsigned integer 8;
option ipxe.aoe code 18 = unsigned integer 8;
option ipxe.http code 19 = unsigned integer 8;
option ipxe.https code 20 = unsigned integer 8;
option ipxe.tftp code 21 = unsigned integer 8;
option ipxe.ftp code 22 = unsigned integer 8;
option ipxe.dns code 23 = unsigned integer 8;
option ipxe.bzimage code 24 = unsigned integer 8;
option ipxe.multiboot code 25 = unsigned integer 8;
option ipxe.slam code 26 = unsigned integer 8;
option ipxe.srp code 27 = unsigned integer 8;
option ipxe.nbi code 32 = unsigned integer 8;
option ipxe.pxe code 33 = unsigned integer 8;
option ipxe.elf code 34 = unsigned integer 8;
option ipxe.comboot code 35 = unsigned integer 8;
option ipxe.efi code 36 = unsigned integer 8;
option ipxe.fcoe code 37 = unsigned integer 8;
option ipxe.vlan code 38 = unsigned integer 8;
option ipxe.menu code 39 = unsigned integer 8;
option ipxe.sdi code 40 = unsigned integer 8;
option ipxe.nfs code 41 = unsigned integer 8;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.200 192.168.1.254;
    option routers 192.168.1.1;
    option subnet-mask 255.255.255.0;
    option broadcast-address 192.168.1.255;
    default-lease-time 600;
    max-lease-time 7200;
}

host win7-pxeboot {
    hardware ethernet xx:xx:xx:xx:xx:xx; 
    fixed-address 192.168.1.201;

    if exists user-class and option user-class = "iPXE" {
        filename "";
        option root-path "iscsi:192.168.1.5::::iqn.2018-08.local.mylab:storage.target00";
        option ipxe.keep-san 1;
        option ipxe.no-pxedhcp 1;
    } else {
        filename "pxeboot/undionly.kpxe";
    }
}
```
```
- \# systemctl start dhcpd
```

## Install TFTP Server
```
- \# yum -y install tftp-server 
- \# cd /var/lib/tftpboot/
- \# mkdir pxeboot
- \# cd pxeboot
- \# wget http://boot.ipxe.org/undionly.kpxe
- \# systemctl start tftp
```

## Install targetcli
```
- \# yum -y install targetcli
- \# targetcli
- \# cd /backstores/fileio
- \# create win7\_32 /winpool/win7\_32.img 30G
- \# cd /iscsi
- \# create iscsi:192.168.1.5::::iqn.2018-08.local.mylab:storage.target00
- \# cd iscsi:192.168.1.5::::iqn.2018-08.local.mylab:storage.target00/tpg1/portals
- \# create 0.0.0.0
- \# cd iscsi:192.168.1.5::::iqn.2018-08.local.mylab:storage.target00/tpg1/luns
- \# create /backstores/fileio/win7\_32
- \# cd iscsi:192.168.1.5::::iqn.2018-08.local.mylab:storage.target00/tpg1/acls
- \# create iqn.2010-04.org.ipxe:ae9e2c81-5264-11cb-a57f-ee7146ed0767
  - Write iqn of the initiator.
  - Once you boot pxeboot, the iqn number is displayed in /var/log/messages.
- \# exit
- \# systemctl start targetcli
```

## Create Windows Installer

### Install AIK
### Write .iso in DVD disk
## Install httpd
### set Windows bootloder
- set boot.sdi bcd 
