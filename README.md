### Red Hat Ansible Automation Platform Infra Using kcli 
#### Baremetal Node Information (TESTED):
* Dell R440 | 40 Core Cpu | 256 GB Ram
* HOST OS: RHEL 8.8 
* Reference: 
  * <https://github.com/karmab/kcli>
  * <https://kcli.readthedocs.io/en/latest/>
* **NOTE: Tested using root permisions only.**
#### Register RHEL Node:
```
subscription-manager register --username _RHN_USERNAME_
```
### Update the RHEL node with latest updates, basic utility packages & ssh key gneration: 
```
dnf update -y
dnf install tmux mc podman-docker bash-completion vim jq tar git  -y
ssh-keygen
```
#### Install libvirt (KVM Virtualization):
```
yum -y install libvirt libvirt-daemon-driver-qemu qemu-kvm
usermod -aG qemu,libvirt $(id -un)
newgrp libvirt
systemctl enable --now libvirtd 
```
#### Configure NTP Server (chrony):
```
dnf install chrony -y

cat <EOF> /etc/chrony.conf
server 0.rhel.pool.ntp.org iburst
server 1.rhel.pool.ntp.org iburst
server 2.rhel.pool.ntp.org iburst
driftfile /var/lib/chrony/drift
makestep 1.0 3
rtcsync
keyfile /etc/chrony.keys
leapsectz right/UTC
logdir /var/log/chrony
bindcmdaddress ::
allow all
EOF

systemctl enable chronyd --now
```
#### Install [kcli](https://kcli.readthedocs.io/en/latest/) - Hybrid Infra Management Toolchain:
```
dnf -y copr enable karmab/kcli
dnf -y install kcli
```
### Create Red Hat Ansible Automation Platform (rhaap) at Red Hat Lab (redhat.lab):
```
kcli create plan redhat.lab -f kcli_plan_RedHatLab_rhaap.yaml -A -P image_url="_RHEL_IMAGE_URL_" -P  rhnuser="_RHN_USER_" -P rhnpassword="_RHN_PASSWORD_"
```
#### Expected execution output:
```
[root@dell-r440-10 RH_AAP_CLUSTER]# kcli list vm
+---------------+--------+-------------+---------+------------+-------------------+
|      Name     | Status |      Ip     |  Source |    Plan    |      Profile      |
+---------------+--------+-------------+---------+------------+-------------------+
| bastion-rhaap |   up   | 10.10.10.10 | rhel-93 | redhat.lab |  rhel-92-c4m8d100 |
|  ctrl-1-rhaap |   up   | 10.10.10.11 | rhel-93 | redhat.lab | rhel-92-c4m16d100 |
|   data-rhaap  |   up   | 10.10.10.50 | rhel-93 | redhat.lab | rhel-92-c4m16d100 |
|  eda-1-rhaap  |   up   | 10.10.10.41 | rhel-93 | redhat.lab | rhel-92-c4m16d100 |
|   ee-1-rhaap  |   up   | 10.10.10.31 | rhel-93 | redhat.lab |  rhel-92-c4m8d100 |
|  node-1-rhaap |   up   | 10.10.10.51 | rhel-93 | redhat.lab |  rhel-92-c4m8d100 |
|  node-2-rhaap |   up   | 10.10.10.52 | rhel-93 | redhat.lab |  rhel-92-c4m8d100 |
|  node-3-rhaap |   up   | 10.10.10.53 | rhel-93 | redhat.lab |  rhel-92-c4m8d100 |
|  pah-1-rhaap  |   up   | 10.10.10.21 | rhel-93 | redhat.lab | rhel-92-c4m16d100 |
+---------------+--------+-------------+---------+------------+-------------------+
[root@dell-r440-10 RH_AAP_CLUSTER]# kcli list plans
+------------+-----------------------------------------------------------------------------------------------------------------+
|    Plan    |                                                       Vms                                                       |
+------------+-----------------------------------------------------------------------------------------------------------------+
| redhat.lab | bastion-rhaap,ctrl-1-rhaap,data-rhaap,eda-1-rhaap,ee-1-rhaap,node-1-rhaap,node-2-rhaap,node-3-rhaap,pah-1-rhaap |
+------------+-----------------------------------------------------------------------------------------------------------------+
[root@dell-r440-10 RH_AAP_CLUSTER]# kcli list network
Listing Networks...
+------------+--------+---------------+------+------------+------+
| Network    |  Type  |      Cidr     | Dhcp |   Domain   | Mode |
+------------+--------+---------------+------+------------+------+
| redhat.lab | routed | 10.10.10.0/24 | True | redhat.lab | nat  |
+------------+--------+---------------+------+------------+------+
[root@dell-r440-10 RH_AAP_CLUSTER]#
```
