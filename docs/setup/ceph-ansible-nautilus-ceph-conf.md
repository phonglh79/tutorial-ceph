## Yêu cầu 

[Đã cài đặt Ceph-Ansible](ceph-ansible-nautilus.md)

## Điều chỉnh ceph.conf 
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
rbd_cache: true

ceph_conf_overrides: 
    global:
        rbd_cache: true
        bluestore_block_db_size: '5737418240'
        bluestore_block_wal_size: '2737418240'
        osd_pool_default_crush_rule: '0'

    mon:
        mon_osd_backfillfull_ratio: '0.95'
        mon_max_pg_per_osd: '500'
        mon_allow_pool_delete: false

    osd:
        osd_crush_chooseleaf_type: '1'
        osd_pool_default_size: '2'
        osd_pool_default_min_size: '1' 
        osd_pool_default_pg_num: '256'
        osd_pool_default_pgp_num: '256'

        ## Scrubbing
        osd_max_scrubs: '1'
        osd_scrub_during_recovery: false
        # osd scrub begin hour: '22' 
        # osd scrub end hour: '4'

        ## Recovery
        # osd recovery threads: '1'
        osd_backfill_scan_max: '16'
        osd_backfill_scan_min: '4'

        ## Backfilling and recovery
        osd_max_backfills: '1'
        osd_recovery_max_active: '1'
        osd_recovery_max_single_start: '1'
        osd_recovery_op_priority: '1'

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
dashboard_admin_password: Cas@2021
grafana_admin_password: Cas@2021

## iSCSI 

EOF
```

Chạy 
```sh 
ansible-playbook site.yml -i inventory_hosts
```

Kết quả 
```sh 
TASK [show ceph status for cluster ceph] ****************************************************************************************************************************************************************************************
Tuesday 23 June 2020  11:28:39 +0700 (0:00:00.892)       0:07:54.821 ********** 
ok: [10.0.12.55 -> 10.0.12.55] => 
  msg:
  - '  cluster:'
  - '    id:     b5e2e377-b01e-4b8e-bb3b-e411543b35ac'
  - '    health: HEALTH_OK'
  - ' '
  - '  services:'
  - '    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 5m)'
  - '    mgr: ceph01(active, since 16s)'
  - '    osd: 6 osds: 6 up (since 4m), 6 in (since 2h)'
  - '    rgw: 3 daemons active (ceph01.rgw0, ceph02.rgw0, ceph03.rgw0)'
  - ' '
  - '  data:'
  - '    pools:   4 pools, 128 pgs'
  - '    objects: 193 objects, 3.9 KiB'
  - '    usage:   6.1 GiB used, 294 GiB / 300 GiB avail'
  - '    pgs:     128 active+clean'
  - ' '
  - '  io:'
  - '    client:   1.2 KiB/s rd, 85 B/s wr, 1 op/s rd, 0 op/s wr'
  - ' '

PLAY RECAP ********************************************************************************************************
10.0.12.55                 : ok=496  changed=19   unreachable=0    failed=0    skipped=484  rescued=0    ignored=0   
10.0.12.56                 : ok=355  changed=13   unreachable=0    failed=0    skipped=378  rescued=0    ignored=0   
10.0.12.57                 : ok=296  changed=14   unreachable=0    failed=0    skipped=337  rescued=0    ignored=0   


INSTALLER STATUS **************************************************************************************************
Install Ceph Monitor           : Complete (0:03:55)
Install Ceph Manager           : Complete (0:00:25)
Install Ceph OSD               : Complete (0:00:41)
Install Ceph RGW               : Complete (0:00:31)
Install Ceph Dashboard         : Complete (0:00:39)
Install Ceph Grafana           : Complete (0:00:28)
Install Ceph Node Exporter     : Complete (0:00:25)

Tuesday 23 June 2020  11:28:39 +0700 (0:00:00.056)       0:07:54.877 ********** 
=============================================================================== 
ceph-handler : restart ceph osds daemon(s) ------------------------------------------------------------------------ 102.65s
ceph-handler : restart ceph mon daemon(s) -------------------------------------------------------------------------- 62.81s
ceph-handler : restart ceph rgw daemon(s) -------------------------------------------------------------------------- 31.18s
ceph-dashboard : set or update dashboard admin username and password ------------------------------------------------ 4.39s
ceph-grafana : wait for grafana to start ---------------------------------------------------------------------------- 4.23s
ceph-common : configure red hat ceph community repository stable key ------------------------------------------------ 2.90s
gather and delegate facts ------------------------------------------------------------------------------------------- 2.43s
ceph-facts : check if the ceph mon socket is in-use ----------------------------------------------------------------- 1.95s
ceph-facts : check if the ceph mon socket is in-use ----------------------------------------------------------------- 1.87s
ceph-config : generate ceph configuration file: ceph.conf ----------------------------------------------------------- 1.85s
ceph-handler : unset noup flag -------------------------------------------------------------------------------------- 1.79s
ceph-config : generate ceph configuration file: ceph.conf ----------------------------------------------------------- 1.76s
ceph-config : look up for ceph-volume rejected devices -------------------------------------------------------------- 1.74s
ceph-config : generate ceph configuration file: ceph.conf ----------------------------------------------------------- 1.73s
ceph-facts : check for a ceph mon socket ---------------------------------------------------------------------------- 1.72s
ceph-dashboard : disable mgr dashboard module (restart) ------------------------------------------------------------- 1.69s
ceph-osd : systemd start osd ---------------------------------------------------------------------------------------- 1.67s
ceph-osd : apply operating system tuning ---------------------------------------------------------------------------- 1.65s
ceph-config : look up for ceph-volume rejected devices -------------------------------------------------------------- 1.58s
ceph-config : look up for ceph-volume rejected devices -------------------------------------------------------------- 1.57s
(venv) [root@ceph01 ceph-ansible]# 
```
