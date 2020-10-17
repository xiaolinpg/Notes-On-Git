### 一、 飞狐浏览器 Filfox 为广大 Filecoin 矿工提供国内种子节点服务
1. 「推荐」直接写入 bootstrapper.pi 文件并编译；

飞狐浏览器 Filfox 为广大 Filecoin 矿工提供国内种子节点服务

2. 使用 lotus net connect 手动连接。

```
lotus net connect /dns4/bootstrap1.testnet.filfox.info/tcp/16666/p2p/12D3KooW9uSxsSh3qwAPxSwwRDVqTTPg8HTBthujVYFXy7Dizb6Q
lotus net connect /dns4/bootstrap2.testnet.filfox.info/tcp/16666/p2p/12D3KooWKths1fzziHsmeMdTdV7dgB9DzoeiGVSwcW2HCygztH9e
```
### 二、关于钱包gas fee设置的讨论
https://filecoinproject.slack.com/archives/C0179RNEMU4/p1601891209040000?thread_ts=1601891209.040000&cid=C0179RNEMU4

### 三、石榴对减少掉算力问题的技术分析
https://mp.weixin.qq.com/s/X_8LcyMj34gXSPBi9fmGlQ

### 四、linux运维常用工具集锦
运维工程师使用的运维平台和工具包括：
Web服务器：apache、tomcat、nginx、lighttpd

监控：nagios、ganglia、cacti、zabbix

自动部署：**ansible**、sshpt、salt

配置管理：puppet、cfengine

负载均衡：lvs、haproxy、nginx

传输工具：scribe、flume

备份工具：rsync、wget

数据库：mysql、oracle、sqlserver

分布式平台：hdfs、mapreduce、spark、storm、hive

分布式数据库：hbase、cassandra、redis、MongoDB

容器：lxc、docker

虚拟化：openstack、xen、kvm

安全：kerberos、selinux、acl、iptables

问题追查：netstat、top、tcpdump、last

### 五、
Magik6k  20 days ago
You can try it now too, but it's a bit tricky:

    Add
```
[patch.crates-io]
storage-proofs = { git = "https://github.com/filecoin-project/rust-fil-proofs" }
filecoin-proofs = { git = "https://github.com/filecoin-project/rust-fil-proofs" }
```
At the bottom of extern/filecoin-ffi/rust/Cargo.toml
```
    FFI_BUILD_FROM_SOURCE=1 RUSTFLAGS='-C target-cpu=native' make clean all lotus-bench
    Run with ... FIL_PROOFS_USE_MULTICORE_SDR=1 taskset -c 0,1,2 ./lotus-worker run ... 
```
(edited)

Note that the worker has to be limited to one CCX / run on cores with shared L3 cache
Magik6k  
You can check that with hwloc-ls
Magik6k 
(This will be automatic when we release this feature, but currently you basically need dedicated PC1 workers to use that)

### 六、GPU 3080
Add the core count like:

BELLMAN_CUSTOM_GPU="GeForce RTX 3080:12345"

peace  20 minutes ago </br>
What does ":12345" do though? @stuberman</br>
stuberman  20 minutes ago </br>
Replace 12345 with the actual number of cores </br>
stuberman  19 minutes ago </br>
The standard GPUs are already set in bellperson with the amount of cores… for a custom GPU you have to tell it how many cores you have </br>
stuberman  16 minutes ago </br>
https://github.com/EntropyPool/bellperson  </br>
EntropyPool/bellperson </br>
Language </br>
Rust </br>
Last updated
