## Yêu cầu 

[Đã cài đặt Ceph-Ansible](ceph-ansible-nautilus.md)

## Bổ sung RGW install 

Bổ sung RGW trên cả 3 node 
- 10.0.12.55
- 10.0.12.56
- 10.0.12.57

Điều chỉnh ceph-ansible 
```sh  
byobu
cd /usr/share/ceph-ansible
source venv/bin/activate
```

```sh 
cat << EOF> group_vars/all.yml
## General 
configure_firewall: False


## Packages

## Install
ceph_origin: repository
ceph_repository: community
ceph_stable_release: nautilus

## Ceph config
public_network: 10.0.12.0/24
cluster_network: 10.0.13.0/24
monitor_interface: eth1

ceph_conf_overrides: {}

### CephFS

### CephRBD 

### CephRGW
radosgw_interface: eth1

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

```sh 
cat <<EOF > inventory_hosts
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

[rgws]
10.0.12.55
10.0.12.56
10.0.12.57

[grafana-server]
10.0.12.56
EOF
```

Chạy 
```sh 
ansible-playbook site.yml -i inventory_hosts
```

Kết quả 
```sh 
TASK [show ceph status for cluster ceph] *****************************************************************************
Tuesday 23 June 2020  10:32:34 +0700 (0:00:00.825)       0:06:09.540 ********** 
ok: [10.0.12.55 -> 10.0.12.55]:> 
  msg:
  - '  cluster:'
  - '    id:     b5e2e377-b01e-4b8e-bb3b-e411543b35ac'
  - '    health: HEALTH_OK'
  - ' '
  - '  services:'
  - '    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 3m)'
  - '    mgr: ceph01(active, since 17s)'
  - '    osd: 6 osds: 6 up (since 3m), 6 in (since 99m)'
  - '    rgw: 3 daemons active (ceph01.rgw0, ceph02.rgw0, ceph03.rgw0)'
  - ' '
  - '  data:'
  - '    pools:   4 pools, 128 pgs'
  - '    objects: 193 objects, 3.9 KiB'
  - '    usage:   6.0 GiB used, 294 GiB / 300 GiB avail'
  - '    pgs:     128 active+clean'
  - ' '
  - '  io:'
  - '    client:   767 B/s rd, 170 B/s wr, 0 op/s rd, 0 op/s wr'
  - ' '

PLAY RECAP **********************************************************************************************************
10.0.12.55                 : ok=486  changed=24   unreachable=0    failed=0    skipped=494  rescued=0    ignored=0   
10.0.12.56                 : ok=347  changed=19   unreachable=0    failed=0    skipped=386  rescued=0    ignored=0   
10.0.12.57                 : ok=287  changed=18   unreachable=0    failed=0    skipped=346  rescued=0    ignored=0   


INSTALLER STATUS ****************************************************************************************************
Install Ceph Monitor           : Complete (0:01:46)
Install Ceph Manager           : Complete (0:00:26)
Install Ceph OSD               : Complete (0:00:40)
Install Ceph RGW               : Complete (0:00:34)
Install Ceph Dashboard         : Complete (0:00:39)
Install Ceph Grafana           : Complete (0:00:28)
Install Ceph Node Exporter     : Complete (0:00:26)

Tuesday 23 June 2020  10:32:34 +0700 (0:00:00.041)       0:06:09.581 ********** 
=============================================================================== 
ceph-handler : restart ceph mon daemon(s) --------------------------------------------------------------------------- 62.76s
ceph-common : install redhat ceph packages -------------------------------------------------------------------------- 21.65s
ceph-dashboard : set or update dashboard admin username and password ------------------------------------------------- 4.33s
ceph-grafana : wait for grafana to start ----------------------------------------------------------------------------- 4.22s
ceph-handler : restart ceph osds daemon(s) --------------------------------------------------------------------------- 4.06s
ceph-common : configure red hat ceph community repository stable key ------------------------------------------------- 2.89s
gather and delegate facts -------------------------------------------------------------------------------------------- 2.40s
ceph-config : look up for ceph-volume rejected devices --------------------------------------------------------------- 1.86s
ceph-config : generate ceph configuration file: ceph.conf ------------------------------------------------------------ 1.81s
ceph-dashboard : disable mgr dashboard module (restart) -------------------------------------------------------------- 1.76s
ceph-config : generate ceph configuration file: ceph.conf ------------------------------------------------------------ 1.74s
ceph-config : generate ceph configuration file: ceph.conf ------------------------------------------------------------ 1.73s
ceph-osd : systemd start osd ----------------------------------------------------------------------------------------- 1.72s
ceph-handler : unset noup flag --------------------------------------------------------------------------------------- 1.69s
ceph-osd : apply operating system tuning ----------------------------------------------------------------------------- 1.68s
ceph-config : look up for ceph-volume rejected devices --------------------------------------------------------------- 1.63s
ceph-validate : read information about the devices ------------------------------------------------------------------- 1.60s
ceph-handler : copy rgw restart script ------------------------------------------------------------------------------- 1.57s
ceph-config : look up for ceph-volume rejected devices --------------------------------------------------------------- 1.57s
ceph-facts : check for a ceph mon socket ----------------------------------------------------------------------------- 1.57s
(venv) [root@ceph01 ceph-ansible]# 
```
