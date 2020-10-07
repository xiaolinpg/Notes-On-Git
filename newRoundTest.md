*812机*<br>
**to do:**<br>
1. 建立三个软raid0，分别用四块，四块，两块硬盘，两块硬盘组存链数据，其它两个缓存；
2. 设置两块缓存盘、一块链数据盘，其余的单盘用做存储；
> 原始状况：
> #cat /proc/mdstat
> Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
> md125 : inactive sdj[2](S) sdk[1](S)
>      15627788976 blocks super 1.2      
> md126 : inactive sdl[6](S)
>      7813894488 blocks super 1.2     
> md127 : inactive sde[1](S) sdh[0](S)
>      15627788976 blocks super 1.2     
> md0 : inactive sdi[3](S) sdg[2](S) sdf[6](S) sdd[4](S)
>      31255577952 blocks super 1.2     
> unused devices: <none>

###操作：# <br>
**参考材料：** [Ubuntu 上创建常用磁盘阵列](https://www.jianshu.com/p/9a458510593a)

```
mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 --chunk=64 /dev/sdb /dev/sdc
mdadm --create --verbose /dev/md101 --level=0 --raid-devices=4 --chunk=64 /dev/sd{d,e,f,g}
mdadm --create --verbose /dev/md102 --level=0 --raid-devices=4 --chunk=64 /dev/sd{h,i,j,k}

```
```
root@worker812:~# lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
NAME      SIZE FSTYPE            TYPE  MOUNTPOINT
loop0    96.6M squashfs          loop  /snap/core/9804
loop1    97.1M squashfs          loop  /snap/core/9993
sda     223.6G                   disk  
├─sda1    512M vfat              part  /boot/efi
└─sda2  223.1G ext4              part  /
sdb       7.3T linux_raid_member disk  
└─md0    14.6T                   raid0 
sdc       7.3T linux_raid_member disk  
└─md0    14.6T                   raid0 
sdd       7.3T linux_raid_member disk  
└─md101  29.1T                   raid0 
sde       7.3T linux_raid_member disk  
└─md101  29.1T                   raid0 
sdf       7.3T linux_raid_member disk  
└─md101  29.1T                   raid0 
sdg       7.3T linux_raid_member disk  
└─md101  29.1T                   raid0 
sdh       7.3T linux_raid_member disk  
└─md102  29.1T xfs               raid0 
sdi       7.3T linux_raid_member disk  
└─md102  29.1T xfs               raid0 
sdj       7.3T linux_raid_member disk  
└─md102  29.1T xfs               raid0 
sdk       7.3T linux_raid_member disk  
└─md102  29.1T xfs               raid0 
sdl       7.3T ext4              disk  
```
不知道为什么会分出xfs，以后再研究吧。<br>

```
mkfs.ext4 -F /dev/md0
sudo mkdir -p /mnt/md0
mount /dev/md0 /mnt/md0
```
检查是否已具有新的磁盘空间：
```
root@worker812:~# df -h -x devtmpfs -x tmpfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       219G  105G  104G  51% /
/dev/loop0       97M   97M     0 100% /snap/core/9804
/dev/loop1       98M   98M     0 100% /snap/core/9993
/dev/sda1       511M  6.1M  505M   2% /boot/efi
/dev/md0         15T   19M   14T   1% /mnt/md0
```
另两个同样，全挂在mnt下
```
mkfs.ext4 -F /dev/md101
mkdir -p /mnt/md101
mount /dev/md101 /mnt/md101
```
```
mkfs.ext4 -F /dev/md102
mkdir -p /mnt/md102
mount /dev/md102 /mnt/md102
```
调整 /etc/mdadm/mdadm.conf 配置使其开机载入
```
mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```
更新 initfamfs 或者初始化RAM文件系统，阵列会在启动前就可以生效：
```
update-initramfs -u
```
在 /etc/fstab 配置文件内加入自动挂载设置：
```
echo '/dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
echo '/dev/md101 /mnt/md101 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
echo '/dev/md101 /mnt/md101 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
```

*关于raid 0 中 chunk的设置参考：*
> he RAID chunk size should suit the I/O characteristics of the data you're working with. The key here is the size of the average I/O request you're going to place on the RAID array; as a rule of thumb, if you want big I/O requests, you should opt for smaller RAID chunk sizes, and if I/O will be small, you should go for larger chunks.
> 
> For example, if your business works with lots of video or large image files, you will want to ensure maximum throughput. That means you will need to spread data across individual drives as much as possible. For this use case smaller RAID chunk sizes (for example, 512 bytes -- one block -- to 8 KB) fit the bill because you want to take data from one drive while the others seek the next chunks to be read.
> 
> At the other extreme in terms of use cases would be running a database in which the amount of data read on each operation is small, say, up to 4 KB. Here you want a single I/O to be dealt with by one drive with one seek action rather than be spilt between more than one drive and multiple seeks. So, for use cases such as databases and email servers, you should go for a bigger RAID chunk size, say, 64 KB or larger.

3. 调整了一下，把md0 挂载到了 /root目录，原来的/root目录改成 /root_tmp，并把所有文件拷贝进新的root目录<br>
