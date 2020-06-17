## Hướng dẫn cài đặt CEPH sử dụng `ceph-deploy`

### Mục tiêu LAB
- Mô hình này sử dụng 3 server, trong đó:
- Host `ceph01` cài đặt `ceph-ansible`, `ceph-mon`,` ceph-osd`, `ceph-mgr`
- Host `ceph02` cài đặt `ceph-osd`
- Host `ceph03` cài đặt `ceph-osd`
- Mô hình khá cơ bản cho việc áp dụng vào môi trường Product

## Chuẩn bị và môi trường LAB (3 Node)

- CentOS8 - 64 bit
- 03: HDD, trong đó:
    - `sda`: sử dụng để cài OS
    - `vdb`: sử dụng làm OSD (nơi chứa dữ liệu)
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
    + Management: 45.77.77.96/20 
    + Ceph Public + Private: 10.1.96.4/24
- Ceph02
    + Management: 140.82.63.99/20
    + Ceph Public + Private: 10.1.96.5/24
- Ceph03
    + Management: 144.202.8.132/20
    + Ceph Public + Private: 10.1.96.6/24


## Các bước chuẩn bị trên từng Server

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

- Đặt IP cho các node
```sh 
cat <<EOF >> /etc/sysconfig/network-scripts/ifcfg-ens7

TYPE=Ethernet
DEVICE=ens7
NAME=ens3
ONBOOT=yes
BOOTPROTO=static
IPADDR=10.1.96.4
PREFIX=24
EOF
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


