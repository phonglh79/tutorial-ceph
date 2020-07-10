## Hướng dẫn cài đặt CEPHAIO sử dụng `ceph-deploy`

### Mục tiêu LAB
- Mô hình này sử dụng 1 server cài đặt CephAIO
- Host `cephaio` cài đặt `ceph-deploy`, `ceph-mon`,` ceph-osd`, `ceph-mgr`

## Chuẩn bị và môi trường LAB

- OS
- CentOS7 - 64 bit
- 05: HDD, trong đó:
- `sda`: sử dụng để cài OS
- `sdb`,`sdc`,`sdd`: sử dụng làm OSD (nơi chứa dữ liệu)
- 03 NICs: 
- `eth0`: dùng để ssh và tải gói cài đặt
- `eth1`: dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào
- `eth2`: dùng để đồng bộ dữ liệu giữa các OSD

- Phiên bản cài đặt : Ceph luminous


## Mô hình 
- Sử dụng mô hình

![](../../images/topol_aio.png)


## IP Planning
- Phân hoạch IP cho các máy chủ trong mô hình trên

![](../../images/ip-planning1_aio.png)


## Các bước chuẩn bị

- Cài đặt NTPD 
```sh 
yum install chrony -y 
```

- Enable NTPD 
```sh 
systemctl enable --now chronyd 
```

- Kiểm tra chronyd hoạt động 
```sh 
chronyc sources -v 
```

- Set hwclock
```sh
hwclock --systohc
```


- Đặt hostname
```sh
hostnamectl set-hostname cephaio
```

- Đặt IP cho các node
```sh 
systemctl disable NetworkManager
systemctl enable network
systemctl start network

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


- Bổ sung file hosts
```sh
cat << EOF >> /etc/hosts
10.10.13.61 cephaio
EOF
```
> Lưu ý network setup trong /etc/hosts chính là đường `eth1` dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào

- Kiểm tra kết nối
```sh 
ping -c 10 cephaio
```

- Khởi động lại máy
```sh
init 6
```

- Bổ sung user `cephuser`
```sh 
sudo useradd -d /home/cephuser -m cephuser
sudo passwd cephuser
````

- Cấp quyền sudo cho `cephuser`
```sh
echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
sudo chmod 0440 /etc/sudoers.d/cephuser
```

## Cài đặt Ceph thực hiện trên cephaio

Bổ sung repo 
```sh 
cat <<EOF> /etc/yum.repos.d/ceph.repo
[ceph]
name=Ceph packages for $basearch
baseurl=https://download.ceph.com/rpm-luminous/el7/x86_64/
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-noarch]
name=Ceph noarch packages
baseurl=https://download.ceph.com/rpm-luminous/el7/noarch
enabled=1
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc

[ceph-source]
name=Ceph source packages
baseurl=https://download.ceph.com/rpm-luminous/el7/SRPMS
enabled=0
priority=2
gpgcheck=1
gpgkey=https://download.ceph.com/keys/release.asc
EOF

yum update -y
```

- Cài đặt `python-setuptools`
```sh 
yum install python-setuptools -y
```

- Cài đặt `ceph-deploy`
```sh 
yum install ceph-deploy -y
```

- Kiểm tra cài đặt 
```sh 
ceph-deploy --version
```
>Kết quả như sau là đã cài đặt thành công ceph-deploy
```sh 
2.0.1
```

- Tạo ssh key 
```sh
ssh-keygen
```
> Bấm ENTER khi có requirement 


- Cấu hình user ssh cho ceph-deploy
```sh 
cat <<EOF> /root/.ssh/config
Host cephaio
    Hostname cephaio
    User cephuser
EOF
```

- Copy ssh key sang
```sh
ssh-copy-id cephaio
```

- Tạo các thư mục `ceph-deploy` để thao tác cài đặt vận hành Cluster
```sh
mkdir /ceph-deploy && cd /ceph-deploy
```

- Khởi tại file cấu hình cho cụm với node quản lý là `cephaio`
```sh
ceph-deploy new cephaio
```

- Kiểm tra lại thông tin folder `ceph-deploy`
```sh 
[cephuser@cephaio ceph-deploy]# ls -lah
total 12K
drwxr-xr-x   2 root root   75 Jan 31 16:31 .
dr-xr-xr-x. 18 root root  243 Jan 31 16:29 ..
-rw-r--r--   1 root root 2.9K Jan 31 16:31 ceph-deploy-ceph.log
-rw-r--r--   1 root root  195 Jan 31 16:31 ceph.conf
-rw-------   1 root root   73 Jan 31 16:31 ceph.mon.keyring
[cephuser@cephaio ceph-deploy]#
```
Trong đó:

+ `ceph.conf` : file config được tự động khởi tạo
+ `ceph-deploy-ceph.log` : file log của toàn bộ thao tác đối với việc sử dụng lệnh `ceph-deploy`
+ `ceph.mon.keyring` : Key monitoring được ceph sinh ra tự động để khởi tạo Cluster


- Chúng ta sẽ bổ sung thêm vào file `ceph.conf` một vài thông tin cơ bản như sau:
> Chú ý: Public network là đường Ceph-Com trong file quy hoạch 
> - Public network là đường Ceph-Com trong file quy hoạch
> - Cluster network là đường Ceph-Replicate trong file quy hoạch

```sh
cat << EOF >> ceph.conf
osd pool default size = 1
osd pool default min size = 1
osd pool default pg num = 128
osd pool default pgp num = 128

osd crush chooseleaf type = 0

public network = 10.10.13.0/24
cluster network = 10.10.14.0/24
EOF
```
- Bổ sung thêm định nghĩa 
    + `public network` : Đường trao đổi thông tin giữa các node Ceph và cũng là đường client kết nối vào 
    + `cluster network` : Đường đồng bộ dữ liệu
- Bổ sung thêm `default size replicate`
- Bổ sung thêm `default pg num`


- Cài đặt ceph trên toàn bộ các node ceph
```sh
ceph-deploy install --release luminous cephaio
```

- Kiểm tra sau khi cài đặt 
```sh 
sudo ceph -v 
```
> Kết quả như sau là đã cài đặt thành công ceph trên node 
```sh 
ceph version 12.2.9 (9e300932ef8a8916fb3fda78c58691a6ab0f4217) luminous (stable)
```

- Khởi tạo cluster với các node `mon` (Monitor-quản lý) dựa trên file `ceph.conf`
```sh
ceph-deploy mon create-initial
```

- Sau khi thực hiện lệnh phía trên sẽ sinh thêm ra 05 file : `ceph.bootstrap-mds.keyring`, `ceph.bootstrap-mgr.keyring`, `ceph.bootstrap-osd.keyring`, `ceph.client.admin.keyring` và `ceph.bootstrap-rgw.keyring`. Quan sát bằng lệnh `ll -alh`

```sh
[cephuser@cephaio ceph-deploy]# ls -lah
total 348K
drwxr-xr-x   2 root root  244 Feb  1 11:40 .
dr-xr-xr-x. 18 root root  243 Feb  1 11:29 ..
-rw-r--r--   1 root root 258K Feb  1 11:40 ceph-deploy-ceph.log
-rw-------   1 root root  113 Feb  1 11:40 ceph.bootstrap-mds.keyring
-rw-------   1 root root  113 Feb  1 11:40 ceph.bootstrap-mgr.keyring
-rw-------   1 root root  113 Feb  1 11:40 ceph.bootstrap-osd.keyring
-rw-------   1 root root  113 Feb  1 11:40 ceph.bootstrap-rgw.keyring
-rw-------   1 root root  151 Feb  1 11:40 ceph.client.admin.keyring
-rw-r--r--   1 root root  195 Feb  1 11:29 ceph.conf
-rw-------   1 root root   73 Feb  1 11:29 ceph.mon.keyring
```

- Để node `cephaio` có thể thao tác với cluster chúng ta cần gán cho node `cephaio` với quyền admin bằng cách bổ sung cho node này `admin.keying`
```sh  
ceph-deploy admin cephaio
```
> Kiểm tra bằng lệnh 
```sh
[cephuser@cephaio ceph-deploy]# sudo ceph -s
cluster:
    id:     39d1a369-bf54-8907-d49b-490a771ac0e2
    health: HEALTH_OK

services:
    mon: 1 daemons, quorum cephaio
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in

data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

## Khởi tạo MGR

Ceph-mgr là thành phần cài đặt yêu cầu cần khởi tạo từ bản Luminous, có thể cài đặt trên nhiều node hoạt động theo cơ chế `Active-Passive`

- Cài đặt ceph-mgr trên cephaio
```sh
ceph-deploy mgr create cephaio
```

- Kiểm tra cài đặt 
```sh
[cephuser@cephaio ceph-deploy]# sudo ceph -s
cluster:
    id:     39d1a369-bf54-8907-d49b-490a771ac0e2
    health: HEALTH_OK

services:
    mon: 1 daemons, quorum cephaio
    mgr: cephaio(active)
    osd: 0 osds: 0 up, 0 in

    data:
    pools:   0 pools, 0 pgs
    objects: 0  objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:
```

- Ceph-mgr hỗ trợ dashboard để quan sát trạng thái của cluster, Enable mgr dashboard trên host cephaio

```sh
sudo ceph mgr module enable dashboard
sudo ceph mgr services
```

- Truy cập vào mgr dashboard với username và password vừa đặt ở phía trên để kiểm tra
```sh 
http://<ip-cephaio>:7000
```
![](../../images/dashboard-l-aio.png)


## Khởi tạo OSD

Tạo OSD thông qua ceph-deploy tại host cephaio

- Trên cephaio, dùng ceph-deploy để partition ổ cứng OSD, thay `cephaio` bằng hostname của host chứa OSD
```sh
ceph-deploy disk zap cephaio /dev/sdb
```

- Tạo OSD với ceph-deploy
```sh
ceph-deploy osd create --data /dev/sdb cephaio
``` 

- Kiểm tra osd vừa tạo bằng lệnh
```sh
sudo ceph osd tree
``` 

- Kiểm tra ID của OSD bằng lệnh
```sh
lsblk
```

Kết quả:
```sh
sdb                                                                                                     8:112  0   39G  0 disk  
└─ceph--42804049--4734--4a87--b776--bfad5d382114-osd--data--e6346e12--c312--4ccf--9b5f--0efeb61d0144  253:5    0   39G  0 lvm   /var/lib/ceph/osd/ceph-0
```

- Thực hiện thao tác trên tương tự cho ổ `sdc` và `sdd`

- Điều chỉnh Crushmap để có thể Replicate trên OSD thay vì trên HOST 
```sh 
cd /home/cephuser/ceph-deploy/
sudo ceph osd getcrushmap -o crushmap
sudo crushtool -d crushmap -o crushmap.decom
sudo sed -i 's|step choose firstn 0 type osd|step chooseleaf firstn 0 type osd|g' crushmap.decom
sudo crushtool -c crushmap.decom -o crushmap.new
sudo ceph osd setcrushmap -i crushmap.new
```

## Kiểm tra
Thực hiện trên cephaio
- Kiểm tra trạng thái của CEPH sau khi cài
```sh
sudo ceph -s
```

- Kết quả của lệnh trên như sau: 
```sh
cephuser@cephaio:~/my-cluster$ sudo ceph -s
cluster:
    id:     39d1a369-bf54-8907-d49b-490a771ac0e2
    health: HEALTH_OK

services:
    mon: 1 daemons, quorum cephaio
    mgr: cephaio(active)
    osd: 3 osds: 3 up, 3 in

data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 bytes
    usage:   3180 MB used, 116 GB / 119 GB avail
    pgs:     
```

- Nếu có dòng `health HEALTH_OK` thì việc cài đặt đã ok.
