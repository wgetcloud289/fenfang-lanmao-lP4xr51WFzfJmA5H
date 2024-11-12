
转载请注明出处：


　　LVM（Logical Volume Manager，逻辑卷管理器）是一个用于Linux系统的磁盘管理工具。它提供了一种更加灵活的存储管理机制，可以方便地进行磁盘的扩容、缩减、快照以及迁移等操作。


## 基本概念


1. 物理卷（PV）：物理磁盘或分区，如`/dev/sda1`。
2. 卷组（VG）：由一个或多个物理卷组成的集合。
3. 逻辑卷（LV）：从卷组中分配的逻辑磁盘，可以被文件系统格式化并用于存储数据。


## 安装LVM


　　在ubuntu系统可以通过下面的命令进行安装




```
# Ubuntu/Debian  
sudo apt-get install lvm2  
```


## 创建LVM


#### 第一步：创建物理卷（PV）


　　假设有一个新的磁盘`/dev/sdb`，需要先将其初始化为物理卷：




```
sudo pvcreate /dev/sdb  
```


　　应用示例：




```
root@swan2:~# sudo pvcreate /dev/vdb
  Physical volume "/dev/vdb" successfully created.
root@sdwan2:~# vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <96.95 GiB
  PE Size               4.00 MiB
  Total PE              24818
  Alloc PE / Size       12409 / 48.47 GiB
  Free  PE / Size       12409 / 48.47 GiB
  VG UUID               RCjkb6-7ngM-9nss-OWOL-eqMR-9MDp-JCyLjk
```


#### 第二步：创建卷组（VG）


　　创建一个名为`vg_data`的卷组，将新创建的物理卷加入其中：




```
sudo vgcreate vg_data /dev/sdb  
```


#### 第三步：创建逻辑卷（LV）


 　　创建一个名为`lv_data`的逻辑卷，大小为10G：



```
sudo lvcreate -n lv_data -L 10G vg_data  
```


#### 第四步：格式化逻辑卷


 　　对逻辑卷进行格式化，例如使用ext4文件系统：



```
sudo mkfs.ext4 /dev/vg_data/lv_data  
```


#### 第五步：挂载逻辑卷


 　　创建一个挂载点，然后将逻辑卷挂载到该挂载点：


```
mkdir /mnt/data  
sudo mount /dev/vg_data/lv_data /mnt/data  
```







## 扩容LVM


　　假设我们需要将逻辑卷`lv_data`扩展到20G，可以遵循以下步骤：


#### 第一步：增加物理卷


　　假设在物理卷`/dev/sdb`上增加了空间（例如增加了第二个物理卷`/dev/sdc`），首先需要将新的物理卷初始化：




```
sudo pvcreate /dev/sdc  
```


　　然后，将其加入到卷组：





```
sudo vgextend vg_data /dev/sdc  
```


　　应用示例：




```
root@sdwan2:~# sudo vgextend ubuntu-vg /dev/vdb
  Volume group "ubuntu-vg" successfully extended
root@swan2:~# vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               1.09 TiB
  PE Size               4.00 MiB
  Total PE              286961
  Alloc PE / Size       12409 / 48.47 GiB
  Free  PE / Size       274552 / <1.05 TiB
  VG UUID               RCjkb6-7ngM-9nss-OWOL-eqMR-9MDp-JCyLjk
   
```


#### 第二步：扩展逻辑卷


 　　使用以下命令将逻辑卷`lv_data`扩展到20G：



```
sudo lvextend -L 20G /dev/vg_data/lv_data  
```


　　或者，如果想使用所有可用的空间：





```
sudo lvextend -l +100%FREE /dev/vg_data/lv_data  
```


　　应用示例：




```
root@swan2:~# lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from 48.47 GiB (12409 extents) to 1.09 TiB (286961 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
root@sdwan2:~#
```


#### 第三步：扩展文件系统


 　　扩展完逻辑卷后，需要扩展文件系统以利用新增的空间：



```
sudo resize2fs /dev/vg_data/lv_data  
```


　　应用示例：




```
root@swan2:~# sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 7, new_desc_blocks = 141
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 293848064 (4k) blocks long.

root@swan2:~#
```


## 查看LVM的信息


　　可以使用以下命令查看LVM的信息：


* 查看所有物理卷：


```
sudo pvdisplay  
```


![](https://img2024.cnblogs.com/blog/1110857/202411/1110857-20241112000829083-824026049.png)
* 查看所有卷组：


```
sudo vgdisplay  
```


![](https://img2024.cnblogs.com/blog/1110857/202411/1110857-20241112000908624-60364710.png)
* 查看所有逻辑卷：


```
sudo lvdisplay  
```


![](https://img2024.cnblogs.com/blog/1110857/202411/1110857-20241112000946175-1104651509.png)
* 查看详细的LVM状态：


```
lvs
```


![](https://img2024.cnblogs.com/blog/1110857/202411/1110857-20241112001007331-434395892.png)








 


 本博客参考[veee加速器](https://liuyunzhuge.com)。转载请注明出处！
