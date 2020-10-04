# lotus install notes #
### 在 机房机器上的测试安装：
1. 原本的系统带有旧版本的lotus，没有卸载，打算直接覆盖上去；
2. 原来的lotus clone目录更改名字，重新clone新的目录下来；
3. 没有从github上直接clone，而是先同步代码到自己的coding.net账户，然后从coding上clone下来，这样下载速度飞快；
4. 按照官方说明安装倚赖包：	https://docs.filecoin.io/get-started/lotus/installation/#software-dependencies

	```
	sudo apt update && sudo apt install mesa-opencl-icd ocl-icd-opencl-dev pkg-config build-essential libclang-dev gcc git bzr jq curl -y && sudo apt upgrade -y
	```
	
5. ~~发现系统的apt 源还是默认源，有点慢，暂时不改了；~~ 没能忍，改成了清华的源重新做了一次（忘了留网址，网上搜索即可）；
6. 顺利make ，并make install，就是make的时间比较长大概有二十分钟？
7. （第二天）lotus daemon --import 测试了一下，先导入了官方的snapshot “https://very-temporary-spacerace-chain-snapshot.s3-us-west-2.amazonaws.com/Spacerace_pruned_stateroots_snapshot_latest.car” 速度太慢，强制kill了进程，改为导入本地.car 文件 /glstore01/  .car
8. 本地导入也花了不少时间，超过半个小时了（没具体计算）
9. 出现大量failed to load hamt node:  错误，

```
2020-10-04T06:42:30.174Z	ERROR	statetree	state/statetree.go:157	loading hamt node bafy2bzacecfmk3z2wfwfi3yg6bsrub4sj6sc5feljgs5g33ptztevmjbt4yes failed: failed to load hamt node: blockstore: block not found
2020-10-04T06:42:30.174Z	WARN	sub	sub/incoming.go:299	cannot validate block message; unknown miner or miner that doesn't meet min power in unsynced chain
2020-10-04T06:42:30.204Z	ERROR	statetree	state/statetree.go:157	loading hamt node bafy2bzacecfmk3z2wfwfi3yg6bsrub4sj6sc5feljgs5g33ptztevmjbt4yes failed: failed to load hamt node: blockstore: block not found
2020-10-04T06:42:30.204Z	WARN	sub	sub/incoming.go:299	cannot validate block message; unknown miner or miner that doesn't meet min power in unsynced chain
2020-10-04T06:42:30.220Z	ERROR	statetree	state/statetree.go:157	loading hamt node bafy2bzacecfmk3z2wfwfi3yg6bsrub4sj6sc5feljgs5g33ptztevmjbt4yes failed: failed to load hamt node: blockstore: block not found
2020-10-04T06:42:30.234Z	ERROR	statetree	state/statetree.go:157	loading hamt node bafy2bzacecfmk3z2wfwfi3yg6bsrub4sj6sc5feljgs5g33ptztevmjbt4yes failed: failed to load hamt node: blockstore: block not found
2020-10-04T06:42:30.285Z	ERROR	statetree	state/statetree.go:157	loading hamt node bafy2bzacecfmk3z2wfwfi3yg6bsrub4sj6sc5feljgs5g33ptztevmjbt4yes failed: failed to load hamt node: blockstore: block not found
```
10. lotus sync status :（虽然存在上诉错误但节点程序似乎可以正常执行）
```
Error: chain contained block marked previously as bad ([bafy2bzaceat2rbuw4glgybvjnsyl2nh2dxg2j4dxtpfpom4kgi2umrrapbdai], bafy2bzacecednjpfr7xuk73s7tim7qn3nhdk444bmtqxpai6ueukwwlf4jgq4) (reason: failed to compute lookback tipset state: making vm: failed to load hamt node: blockstore: block not found)
```
**Slack Channel中提供的一些信息：**
```
@Zhao KK  3 days ago
before i export with 'lotus chain export snapshot.car' in a working lotus repo with full synced with lotus 0.8.0, when i import, 'block not found' happens. which version of lotus should i use to export ?

@ribasushi - PLFD  3 days ago
it doesn't matter
what is important is:
- you made the export from a full node ( not one made from a snapshot recently )
- you supplied the --tipset flag with a certain step-back ( at least 10 tipsets )
- ideally you just carry over finality stateroots as above
```
