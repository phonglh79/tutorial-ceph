## Hướng dẫn cài đặt CEPH sử dụng `ceph-ansible`

> Nên cài đặt python3 và ansible trong venv

## Mục tiêu LAB
- Mô hình này sử dụng 3 server, trong đó:
- Host `ceph01` cài đặt `ceph-ansible`, `ceph-mon`,` ceph-osd`, `ceph-mgr`
- Host `ceph02` cài đặt `ceph-osd`, `grafana-server` (Monitor)
- Host `ceph03` cài đặt `ceph-osd`
- Mô hình khá cơ bản cho việc áp dụng vào môi trường Product

## Chuẩn bị và môi trường LAB (3 Node)

- CentOS7.8.2003
- 03: HDD, trong đó:
    - `sda`: sử dụng để cài OS
    - `vdb`,`vdc`: sử dụng làm OSD (nơi chứa dữ liệu)
- 02 NICs: 
    - `eth0`: dùng để ssh và tải gói cài đặt
    - `eth1`: dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào + dùng để đồng bộ dữ liệu giữa các OSD

> Rút gọn 1 NIC so với thực tế 

- Phiên bản cài đặt : Ceph Nautilus

## Mô hình 
- Sử dụng mô hình 3 node 

## IP Planning
- Phân hoạch IP cho các máy chủ trong mô hình trên
- Ceph01
    + Management: 10.0.10.55/24
    + Ceph Public: 10.0.12.55/24
    + Private: 10.0.13.55/24
- Ceph02
    + Management: 10.0.10.56/24
    + Ceph Public: 10.0.12.56/24
    + Private: 10.0.13.56/24
- Ceph03
    + Management: 10.0.10.57/24
    + Ceph Public: 10.0.12.57/24
    + Private: 10.0.13.57/24

## Các bước chuẩn bị trên từng Server

- Đặt IP cho các node
```sh 
echo "Setup IP eth0"
nmcli c modify eth0 ipv4.addresses 10.10.10.61/24
nmcli c modify eth0 ipv4.gateway 10.10.10.1
nmcli c modify eth0 ipv4.dns 8.8.8.8
nmcli c modify eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

echo "Setup IP eth1"
nmcli c modify eth1 ipv4.addresses 10.10.13.61/24
nmcli c modify eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

echo "Setup IP eth2"
nmcli c modify eth2 ipv4.addresses 10.10.14.61/24
nmcli c modify eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes
```

- Cài đặt epel-relese và update OS 
```sh
yum install epel-release -y
yum update -y
```

- Cài đặt CMD_log 
```sh 
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | bash
```

- Cài đặt NTPD 
```sh 
yum install chrony -y 
```

- Enable NTPD 
```sh 
systemctl start chronyd 
systemctl enable chronyd 
```

- Cấu hình timezone 
```sh 
timedatectl set-timezone Asia/Ho_Chi_Minh
```

- Kiểm tra chronyd hoạt động 
```sh 
chronyc sources -v 
timedatectl
```

- Set hwclock 
```sh 
hwclock --systohc
```

- Đặt hostname
```sh
hostnamectl set-hostname ceph01
```

- Vô hiệu hóa Selinux
```sh
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

- Mở port cho Ceph trên Firewalld  
```sh 
# start enable
systemctl start firewalld
systemctl enable firewalld

# ceph-ansible
sudo firewall-cmd --zone=public --add-port=80/tcp --permanent
sudo firewall-cmd --zone=public --add-port=2003/tcp --permanent
sudo firewall-cmd --zone=public --add-port=4505-4506/tcp --permanent
sudo firewall-cmd --reload

# mon
sudo firewall-cmd --zone=public --add-port=6789/tcp --permanent
sudo firewall-cmd --reload

# osd
sudo firewall-cmd --zone=public --add-port=6800-7300/tcp --permanent
sudo firewall-cmd --reload

# rgw
sudo firewall-cmd --zone=public --add-port=7480/tcp --permanent
sudo firewall-cmd --reload

# mds
sudo firewall-cmd --zone=public --add-port=6800/tcp --permanent
sudo firewall-cmd --reload
```

- Hoặc có thể disable firewall 
```sh 
sudo systemctl disable firewalld
sudo systemctl stop firewalld
```


## Cài đặt python3 trên node admin 

- Cài đặt python3 
```sh 
sudo yum install centos-release-scl
yum update -y 
sudo yum install python36 -y
```

- Cài đặt pip3
```sh 
yum install wget -y
wget https://bootstrap.pypa.io/get-pip.py
sudo python3.6 get-pip.py
```

- Cài đặt venv
```sh 
sudo pip install virtualenv
```

## Cài đặt ansible trên node admin 

Cài đặt 
```sh 
sudo yum install ansible -y 
# pip intall ansible=="2.8"
```

## Cấu hình ceph-ansible 

Clone repo 
```sh 
yum install git -y 
git clone https://github.com/ceph/ceph-ansible.git
```

Kiểm tra các bản ceph-ansible hỗ trợ 
```sh 
cd ceph-ansible 
git checkout stable-4.0
mv ceph-ansible /usr/share/ceph-ansible
ln -s /usr/share/ceph-ansible/group_vars /etc/ansible/group_vars
```

Các bản hỗ trợ bao gồm 
- Stable 3.0 Jewel và Luminous yêu cầu Ansible 2.4 
- Stable 3.1 Luminous và Mimic yêu cầu Ansible 2.4 
- Stable 3.2 Luminous và Mimic yêu cầu Ansible 2.6 
- Stable 4.0 Nautilus yêu cầu Ansible 2.8  

Cài đặt các requirement trong venv
```sh 
cd ceph-ansible /usr/share/ceph-ansible
virtualenv venv -p python3.6
source venv/bin/activate
pip install -r requirements.txt
```

Tạo file inventory `/usr/share/ceph-ansible/inventory_hosts` để description tất cả các server
```sh 
[mons]
10.0.12.55
10.0.12.56
10.0.12.57

[osds]
10.0.12.55
10.0.12.56
10.0.12.57

[mgrs]
10.0.12.55

[grafana-server]
10.0.12.56
``` 

Tạo ssh-key 
```sh 
ssh-keygen
```

Copy ssh key qua các node 
```sh 
ssh-copy-id 10.0.12.56
ssh-copy-id 10.0.12.57
ssh-copy-id 10.0.12.58
```

Kiểm tra 
```sh 
ansible -m ping all 
```

Kêt quả 
```sh 
(venv) [root@ceph01 ceph-ansible]# ansible -m ping -i inventory_hosts all
[DEPRECATION WARNING]: The TRANSFORM_INVALID_GROUP_CHARS settings is set to allow bad characters in group names by default,
this will change, but still be user configurable on deprecation. This feature will be removed in version 2.10. Deprecation
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details

10.0.12.57 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.0.12.56 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
10.0.12.55 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
(venv) [root@ceph01 ceph-ansible]#
```

Cấu hình fil all.yaml
```sh 
cp group_vars/{all.yml.sample,all.yml}
> group_vars/all.yml
cat << EOF>> group_vars/all.yml
## Packages

## Install
ceph_origin: repository
ceph_repository: community
ceph_stable_release: nautilus
monitor_interface: eth1
public_network: 10.0.12.0/24
cluster_network: 10.0.13.0/24

## Ceph config

## CephFS

## NFS-Ganesha

## Multisite -RGW

## Config override

## OS turning

## Docker

## Openstack 

## Dashboard - Grafana 
dashboard_admin_password: Cas@2020
grafana_admin_password: Cas@2020

## iSCSI 

EOF
```

Cấu hình osds.yml
```sh 
cp group_vars/{osds.yml.sample,osds.yml}
> group_vars/osds.yml
cat << EOF>> group_vars/osds.yml
osd_scenario: non-collocated
osd_objectstore: bluestore
devices:
  - /dev/sdb
  - /dev/sdc
EOF
```

Cấu hình site.yml 
```sh 
cp {site.yml.sample,site.yml}
```

```sh 
  - mons
  - osds
  - mdss
  - rgws
  - nfss
  - rbdmirrors
  - clients
  - mgrs
  - iscsigws
  - iscsi-gws # for backward compatibility only!
  - grafana-server
  - rgwloadbalancers
```

## Deploy 
Cài đặt byobu 
```sh 
yum install byobu -y 
```

Deploy ceph
```sh 
byobu
cd /usr/share/ceph-ansible
source venv/bin/activate
ansible-playbook site.yml -i inventory_hosts
```

Log install OK
```sh 
...

PLAY RECAP *********************************************************************************************************
10.0.12.55                 : ok=385  changed=52   unreachable=0    failed=0    skipped=498  rescued=0    ignored=0
10.0.12.56                 : ok=278  changed=56   unreachable=0    failed=0    skipped=375  rescued=0    ignored=0
10.0.12.57                 : ok=215  changed=33   unreachable=0    failed=0    skipped=336  rescued=0    ignored=0


INSTALLER STATUS ****************************************************************************************************
Install Ceph Monitor           : Complete (0:00:36)
Install Ceph Manager           : Complete (0:00:47)
Install Ceph OSD               : Complete (0:00:55)
Install Ceph Dashboard         : Complete (0:00:32)
Install Ceph Grafana           : Complete (0:00:46)
Install Ceph Node Exporter     : Complete (0:00:50)

Monday 22 June 2020  01:00:39 +0700 (0:00:00.045)       2:06:44.256 ***********
=====================================================================================
ceph-infra : open ceph networks on monitor ---------------------------------- 7201.02s
ceph-common : install redhat ceph packages ---------------------------------- 70.22s
ceph-container-engine : install container packages -------------------------- 24.42s
ceph-mgr : install ceph-mgr packages on RedHat or SUSE ---------------------- 17.25s
ceph-grafana : wait for grafana to start ------------------------------------ 17.20s
ceph-infra : install firewalld python binding ------------------------------- 12.73s
ceph-osd : wait for all osd to be up ---------------------------------------- 11.54s
ceph-osd : use ceph-volume lvm batch to create bluestore osds --------------- 8.54s
ceph-mgr : wait for all mgr to be up ---------------------------------------- 6.22s
ceph-grafana : install ceph-grafana-dashboards package on RedHat or SUSE ---- 5.31s
ceph-dashboard : set or update dashboard admin username and password -------- 4.43s
ceph-common : install yum plugin priorities --------------------------------- 3.69s
ceph-mon : waiting for the monitor(s) to form the quorum... ----------------- 3.56s
ceph-common : configure red hat ceph community repository stable key -------- 2.90s
ceph-mon : fetch ceph initial keys ------------------------------------------ 2.79s
gather and delegate facts --------------------------------------------------- 2.51s
ceph-container-engine : start container service ----------------------------- 2.50s
ceph-mgr : disable ceph mgr enabled modules --------------------------------- 2.38s
ceph-mgr : add modules to ceph-mgr ------------------------------------------ 2.12s
ceph-config : look up for ceph-volume rejected devices ---------------------- 1.69s
(venv) [root@ceph01 ceph-ansible]#
```

Kiểm tra ceph
```sh 
[root@ceph01 ~]# ceph -v
ceph version 14.2.9 (581f22da52345dba46ee232b73b990f06029a2a0) nautilus (stable)
[root@ceph01 ~]# ceph -s
  cluster:
    id:     9889fc17-5f1e-43c4-a631-99009e12dfe2
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 5h)
    mgr: ceph01(active, since 5h)
    osd: 6 osds: 6 up (since 5h), 6 in (since 5h)

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   6.0 GiB used, 294 GiB / 300 GiB avail
    pgs:

[root@ceph01 ~]# 
```

## Tài liệu tham khảo 

- https://docs.ceph.com/ceph-ansible/master/
- https://www.marksei.com/how-to-install-ceph-with-ceph-ansible/ (*)
- https://kruschecompany.com/ceph-ansible/
