## RBD Mirror
Mô hình 

- 2 node Pro

![](../images/rbd-mirror/mirror.png)

Create pool
```sh 
ceph osd pool create rbd 256
ceph osd pool set rbd size 1
```

Enable 
```sh 
rbd mirror pool enable {pool-name} {mode}
```

Enable mirror trên pool `rbd`
```sh 
rbd mirror pool enable rbd pool
```

> Option `pool` hoặc `image`


Disable mirror
```sh 
rbd mirror pool disable {pool-name}
```

VD: 
```sh 
rbd mirror pool disable rbd
```

Add cluster muốn kết nối peer 
```sh 
rbd mirror pool peer add {pool-name} {client-name}@{cluster-name}
``