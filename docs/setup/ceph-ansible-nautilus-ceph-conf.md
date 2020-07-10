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
        osd pool default crush rule: '0'

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
dashboard_admin_password: Cas@2020
grafana_admin_password: Cas@2020

## iSCSI 

EOF
```

Chạy 
```sh 
ansible-playbook site.yml -i inventory_hosts
```

Kết quả 
```sh 
TASK [show ceph status for cluster ceph] ******************************************************************************
Tuesday 23 June 2020  14:16:07 +0700 (0:00:00.837)       0:05:02.877 ********** 
ok: [10.0.12.55 -> 10.0.12.55] => 
  msg:
  - '  cluster:'
  - '    id:     05104fd8-9b88-4810-b27a-8a3fc54509c8'
  - '    health: HEALTH_OK'
  - ' '
  - '  services:'
  - '    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 3m)'
  - '    mgr: ceph01(active, since 12s)'
  - '    osd: 6 osds: 6 up (since 2m), 6 in (since 5m)'
  - ' '
  - '  data:'
  - '    pools:   0 pools, 0 pgs'
  - '    objects: 0 objects, 0 B'
  - '    usage:   6.0 GiB used, 294 GiB / 300 GiB avail'
  - '    pgs:     '
  - ' '

PLAY RECAP *********************************************************************************************************
10.0.12.55                 : ok=380  changed=13   unreachable=0    failed=0    skipped=457  rescued=0    ignored=0   
10.0.12.56                 : ok=267  changed=11   unreachable=0    failed=0    skipped=341  rescued=0    ignored=0   
10.0.12.57                 : ok=211  changed=9    unreachable=0    failed=0    skipped=296  rescued=0    ignored=0   


INSTALLER STATUS ***************************************************************************************************
Install Ceph Monitor           : Complete (0:01:45)
Install Ceph Manager           : Complete (0:00:24)
Install Ceph OSD               : Complete (0:00:40)
Install Ceph Dashboard         : Complete (0:00:32)
Install Ceph Grafana           : Complete (0:00:28)
Install Ceph Node Exporter     : Complete (0:00:26)

Tuesday 23 June 2020  14:16:07 +0700 (0:00:00.055)       0:05:02.932 ********** 
=============================================================================== 
ceph-handler : restart ceph mon daemon(s) --------------------------------------------------------------------------- 62.85s
ceph-dashboard : set or update dashboard admin username and password ------------------------------------------------- 4.21s
ceph-grafana : wait for grafana to start ----------------------------------------------------------------------------- 4.21s
ceph-handler : restart ceph osds daemon(s) --------------------------------------------------------------------------- 4.15s
gather and delegate facts -------------------------------------------------------------------------------------------- 2.95s
ceph-common : configure red hat ceph community repository stable key ------------------------------------------------- 2.70s
ceph-infra : install chrony ------------------------------------------------------------------------------------------ 2.12s
ceph-config : look up for ceph-volume rejected devices --------------------------------------------------------------- 1.88s
ceph-osd : apply operating system tuning ----------------------------------------------------------------------------- 1.69s
ceph-handler : unset noup flag --------------------------------------------------------------------------------------- 1.69s
ceph-osd : systemd start osd ----------------------------------------------------------------------------------------- 1.67s
ceph-osd : unset noup flag ------------------------------------------------------------------------------------------- 1.65s
ceph-facts : resolve device link(s) ---------------------------------------------------------------------------------- 1.54s
ceph-config : look up for ceph-volume rejected devices --------------------------------------------------------------- 1.54s
ceph-config : look up for ceph-volume rejected devices --------------------------------------------------------------- 1.51s
ceph-handler : check if the ceph osd socket is in-use ---------------------------------------------------------------- 1.31s
ceph-handler : check for a ceph mon socket --------------------------------------------------------------------------- 1.30s
ceph-facts : get default crush rule value from ceph configuration ---------------------------------------------------- 1.26s
ceph-handler : check if the ceph osd socket is in-use ---------------------------------------------------------------- 1.22s
ceph-osd : set noup flag --------------------------------------------------------------------------------------------- 1.22s
(venv) [root@ceph01 ceph-ansible]# 
```

Kiểm tra `ceph.conf`
```sh 


```
