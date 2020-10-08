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

### 操作： <br>
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
	这样.lotus文件夹就放在新的md0阵列上

## Game Starting

```
Variables common to most Lotus binaries:

    LOTUS_FD_MAX: Sets the file descriptor limit for the process
    LOTUS_JAEGER: Sets the Jaeger URL to send traces. See TODO.
    LOTUS_DEV: Any non-empty value will enable more verbose logging, useful only for developers.
    GOLOG_OUTPUT: Controls where the program logs. Possible values: stdout, stderr, file. Multiple values can be combined with '+'.
    GOLOG_FILE: Path to file to log to.
    GOLOG_LOG_FMT: Logging format (json, nocolor).
```
```
Variables specific to the Lotus daemon:
    LOTUS_PATH: Location to store Lotus data (defaults to ~/.lotus).
    LOTUS_SKIP_GENESIS_CHECK=_yes_: Set only if you wish to run a lotus network with a different genesis block.
    LOTUS_CHAIN_TIPSET_CACHE: Sets the size for the chainstore tipset cache. Defaults to 8192. Increase if you perform frequent arbitrary tipset lookups.
    LOTUS_CHAIN_INDEX_CACHE: Sets the size for the epoch index cache. Defaults to 32768. Increase if you perform frequent deep chain lookups for block heights far from the latest height.
    LOTUS_BSYNC_MSG_WINDOW: Sets the initial maximum window size for message fetching blocksync request. Set to 10-20 if you have an internet connection with low bandwidth
```
修改env（编辑/etc/profile文件）：

```
root@worker812:~/lotus# env
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
SSH_CONNECTION=111.203.200.194 4511 172.28.8.12 22
LESSCLOSE=/usr/bin/lesspipe %s %s
LANG=en_US.UTF-8
IPFS_GATEWAY=https://proof-parameters.s3.cn-south-1.jdcloud-oss.com/ipfs/
RUST_LOG=Trace
OLDPWD=/root/lotus/build
GOPROXY=https://goproxy.cn
FFI_BUILD_FROM_SOURCE=1
XDG_SESSION_ID=2
USER=root
FIL_PROOFS_USE_GPU_COLUMN_BUILDER=1
FIL_PROOFS_USE_GPU_TREE_BUILDER=1
PWD=/root/lotus
HOME=/root
SSH_CLIENT=111.203.200.194 4511 22
FIL_PROOFS_PARENT_CACHE=/mnt/md102/parent_proof
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
FIL_PROOFS_USE_MULTICORE_SDR=1
BELLMAN_CPU_UTILIZATION=0.875
TMPDIR=/mnt/md101/TMPDIR
WORKER_PATH=/root/.lotusworker
LOTUS_PATH=/root/.lotus
FIL_PROOFS_MAXIMIZE_CACHING=1
SSH_TTY=/dev/pts/0
MAIL=/var/mail/root
TERM=xterm-256color
SHELL=/bin/bash
LOTUS_MINER_PATH=/root/.miner_storage
SHLVL=1
LOGNAME=root
XDG_RUNTIME_DIR=/run/user/0
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/root/.cargo/bin:/usr/local/mfs/sbin:/usr/local/mfs/bin
FIL_PROOFS_PARAMETER_CACHE=/mnt/md101/para_proof
RUSTFLAGS=-C target-cpu=native -g
LESSOPEN=| /usr/bin/lesspipe %s
_=/usr/bin/env
```
#### make install-daemon-service
```
make install-daemon-service
make install-miner-service
```
output:
```
root@worker812:~/lotus# make install-daemon-service
go: creating work dir: stat /mnt/md101/TMPDIR: no such file or directory
expr: syntax error
install -C ./lotus /usr/local/bin/lotus
mkdir -p /etc/systemd/system
mkdir -p /var/log/lotus
install -C -m 0644 ./scripts/lotus-daemon.service /etc/systemd/system/lotus-daemon.service
systemctl daemon-reload

lotus-daemon service installed. Don't forget to run 'sudo systemctl start lotus-daemon' to start it and 'sudo systemctl enable lotus-daemon' for it to be enabled on startup.
```
```
root@worker812:~/lotus# make install-miner-service
go: creating work dir: stat /mnt/md101/TMPDIR: no such file or directory
expr: syntax error
install -C ./lotus-miner /usr/local/bin/lotus-miner
install -C ./lotus /usr/local/bin/lotus
mkdir -p /etc/systemd/system
mkdir -p /var/log/lotus
install -C -m 0644 ./scripts/lotus-daemon.service /etc/systemd/system/lotus-daemon.service
systemctl daemon-reload

lotus-daemon service installed. Don't forget to run 'sudo systemctl start lotus-daemon' to start it and 'sudo systemctl enable lotus-daemon' for it to be enabled on startup.
mkdir -p /etc/systemd/system
mkdir -p /var/log/lotus
install -C -m 0644 ./scripts/lotus-miner.service /etc/systemd/system/lotus-miner.service
systemctl daemon-reload

lotus-miner service installed. Don't forget to run 'sudo systemctl start lotus-miner' to start it and 'sudo systemctl enable lotus-miner' for it to be enabled on startup.
```
### wallets
```
root@worker812:~# lotus wallet list
Address                                                                                 Balance  Nonce  Default  
t3u4qx7nwucpqlxjcl2y6bv7x2okfxqwobuu24ozxkvan4ljtxqcebtetq4td3couv6xxxj2eimletd42urb2q  0 FIL    0               
t3uim6m7t4ushu2v4mxtphbdvvigjg6zcvh5f6zgmilepfd3k3wferik7is6g3ufriq3pp2zzwvlkzxxatawja  0 FIL    0      X        
t3wlnfaftlfr6il7mt3fqjunq6dbn4bbq6qefwfxzqop2ub2mxzk6hp5sh27soqqmkdykttz7g2va4kpor4x3a  0 FIL    0               
```
First time run：
```
lotus-miner init --owner=t3u4qx7nwucpqlxjcl2y6bv7x2okfxqwobuu24ozxkvan4ljtxqcebtetq4td3couv6xxxj2eimletd42urb2q --no-local-storage
```
没有按照官方文档加 --worker 地址，因为没给发fil，没有gas可以烧（懒）<br>
设置 --no-local-storage 后，设定miner目录<br>
> https://docs.filecoin.io/mine/lotus/custom-storage-layout/#custom-location-for-sealing

### TIPS
**lotus-daemon 系统服务是好用的，前一次没好用可能是等待的时间不够长，还没有完全启动**

### Miner started

```
config.toml
[Libp2p]
  ListenAddresses = ["/ip4/0.0.0.0/tcp/8086"]
  AnnounceAddresses = ["/ip4/61.155.145.133/tcp/8086"]
```


