```yml
---
dummy:

fetch_directory: fetch/

cluster: ceph

mon_group_name: mons
osd_group_name: osds
rgw_group_name: rgws
mds_group_name: mdss
nfs_group_name: nfss
rbdmirror_group_name: rbdmirrors
client_group_name: clients
iscsi_gw_group_name: iscsigws
mgr_group_name: mgrs
rgwloadbalancer_group_name: rgwloadbalancers
grafana_server_group_name: grafana-server

ceph_repository_type: cdn
ceph_origin: repository
ceph_repository: community
ceph_stable_release: nautilus

generate_fsid: true

cephx: true

rbd_client_admin_socket_path: /var/run/ceph # must be writable by QEMU and allowed by SELinux or AppArmor

monitor_interface: eth0
ip_version: ipv4

journal_size: 5120 # OSD journal size in MB
cluster_network: 172.31.16.0/20
# public_network: 0.0.0.0/0
osd_auto_discovery_exclude: "dm-*|loop*|md*|rbd*"

mds_max_mds: 1

radosgw_address: 172.31.30.12

ceph_conf_overrides:
  global:
    osd_pool_default_pg_num: 8
    osd_pool_default_size: 3
    osd_pool_default_min_size: 0

containerized_deployment: False
timeout_command: "{{ 'timeout --foreground -s KILL ' ~ docker_pull_timeout if (docker_pull_timeout != '0') and (ceph_docker_dev_image is undefined or not ceph_docker_dev_image) else '' }}"

rolling_update: false

docker_pull_timeout: "290s"

dashboard_enabled: True
dashboard_protocol: http
dashboard_port: 8443
dashboard_admin_user: admin
dashboard_admin_password: p@ssw0rd
grafana_admin_user: admin
grafana_admin_password: admin

client_connections: {}

container_exec_cmd:
docker: false
```



Nginx: 144.202.8.50/rR2,]h.[=mvKWep2 10.1.96.6 
Ceph01: 45.77.77.96/-5aAfd$S+Paty=Hv 10.1.96.4 
Ceph02: 144.202.8.132/7F!sKQ*bQ2A4HEX. 10.1.96.5 
Ceph03: 45.77.156.138/,2nQvb)z4)LBBDL5 10.1.96.7 