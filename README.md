

* Red Hat Environment Information 
	* Dell R440 | 40 Core Cpu | 256 GB Ram
	* HOST OS: RHEL 8.8 
	* References: 
		* <https://kcli.readthedocs.io/en/latest/>

* Register RHEL Node:
  
``subscription-manager register --username rhn-support-hdalwadi``

* Update the RHEL node with latest updates, basic utility packages & ssh key gneration: 

``dnf update -y``
``dnf install tmux mc podman-docker bash-completion vim jq tar git  -y``
``ssh-keygen``

* Install libvirt (KVM Virtualization)

``yum -y install libvirt libvirt-daemon-driver-qemu qemu-kvm``
``usermod -aG qemu,libvirt $(id -un)``
``newgrp libvirt``
``systemctl enable --now libvirtd`` 

* Configure NTP Server (chrony)

``dnf install chrony -y``
	
``cat <EOF> /etc/chrony.conf``
``server time.cloudflare.com iburst``
``driftfile /var/lib/chrony/drift``
``makestep 1.0 3``
``rtcsync``
``keyfile /etc/chrony.keys``
``leapsectz right/UTC``
``logdir /var/log/chrony``
``bindcmdaddress ::``
``allow all``
``EOF``

``systemctl enable chronyd --now``

* Install [kcli](https://kcli.readthedocs.io/en/latest/) - Hybrid Infra Management Toolchain 

``dnf -y copr enable karmab/kcli``
``dnf -y install kcli``

* Create Red Hat Ansible Automation Platform (rhaap) at Red Hat Lab (redhat.lab)

``kcli create plan -f kcli_plan_RedHatLab_rhaap.yaml -A -P image_url="_RHEL_IMAGE_URL_" -P  rhnuser="_RHN_USER_" -P rhnpassword="_RHN_PASSWORD_"
