# Storage cơ bản 

- [Các note ghi chép liên quan đến Storage](https://github.com/uncelvel/storage)

# Lý thuyết Ceph Storage, Các services, thành phần của Ceph

- [Ceph - Overview](docs/knowledge_base/ceph-overview.md)
- [Ceph RADOS](docs/knowledge_base/ceph-rados.md)
- [CRUSH](docs/knowledge_base/crush.md)
- [Ceph Storage Backend](docs/knowledge_base/bluestore_vs_filestore.md)
- [Ceph MON - Monitor (ceph-mon)](docs/knowledge_base/ceph-mon.md)
- [Ceph OSD - Object Storage Device (ceph-osd)](docs/knowledge_base/ceph-osd.md)
- [Ceph RBD - RADOS Block Device](docs/knowledge_base/ceph-rbd.md)
- [Ceph MDS - Metadata Server (ceph-mds)](docs/knowledge_base/ceph-mds.md)
- [Ceph RADOSGW - Object Gateway(ceph-radosgw)](docs/knowledge_base/ceph-radosgw.md)
- [Ceph MGR - Manager (ceph-mgr)](docs/knowledge_base/ceph-mgr.md)
- [Thuật toán PAXOS](docs/knowledge_base/paxos.md)
- [Cơ chế xác thực của Ceph](docs/knowledge_base/ceph-authen.md)
- [Các flag của Cluster Ceph](docs/knowledge_base/ceph-flag.md)
- [Các trạng thái của PG](docs/knowledge_base/ceph-pg-status.md)

# Tài liệu cài đặt

Scripts
- [Cài đặt CephAIO bản Luminous sử dụng scripts](https://github.com/uncelvel/script-ceph-lumi-aio)

Ceph-deploy
- [Cài đặt CephAIO bản Luminous manual-cephuser](docs/setup/ceph-luminous-aio.md)
- [Cài đặt Ceph bản Luminous](docs/setup/ceph-luminous.md)
- [Cài đặt Ceph bản Mimic](docs/setup/ceph-mimic.md)
- [Cài đặt Ceph bản Nautilus](docs/setup/ceph-nautilus.md)
- [Cài đặt Ceph-RadosGW HA bản Nautilus](docs/setup/ceph-radosgw.md)

Ceph-Ansible
- [Ceph Ansible Nautilus](docs/setup/ceph-ansible-nautilus.md)
- [Ceph Ansible Nautilus- Cài đặt thêm RGW](docs/setup/ceph-ansible-nautilus-rgw.md)
- [Ceph Ansible Nautilus- Add/Modify ceph config](docs/setup/ceph-ansible-nautilus-ceph-conf.md)
- [Ceph Ansible Nautilus base containerized](docs/setup/ceph-ansible-nautilus-container.md)

# Tài liệu vận hành tích hợp

- [=========> Ceph Cheat sheet <=========](docs/operating/ceph-cheat-sheet.md)

CephRBD
- [Sử dụng RBD (Block Storage) cơ bản](docs/operating/ceph-vs-client-linux.md)
- [Tích hợp Ceph với OpenStack](docs/operating/ceph-vs-openstack.md)
- [RBD Mirror](docs/operating/rbd-mirror.md)

CephFS
- [Sử dụng CephFS (File Storage) cơ bản]()

RadosGW
- [Sử dụng RGW (Object Storage) cơ bản]()
- [radosgw-admin - RADOS gateway user administration utility](https://www.mankier.com/8/radosgw-admin)
- [s3cmd - Command Line S3 Client Software and S3 Backup](docs/operating/s3cmd.md)
- [rgwadmin - Ceph Object Storage Admin API python library bindings.](https://github.com/uncelvel/rgwadmin)
- [s3ync - Sync data S3](https://github.com/uncelvel/s3sync)

# Benchmark & Troubleshooting

- [Node Ceph hỏng](docs/operating/ceph-hardware-crash.md)
- [Note case vận hành](docs/operating/README.md)