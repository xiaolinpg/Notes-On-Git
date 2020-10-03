# lotus install notes #
### 在 机房机器上的测试安装：#
1. 原本的系统带有旧版本的lotus，没有卸载，打算直接覆盖上去；
2. 原来的lotus clone目录更改名字，重新clone新的目录下来；
3. 没有从github上直接clone，而是先同步代码到自己的coding.net账户，然后从coding上clone下来，这样下载速度飞快；
4. 按照官方说明安装倚赖包：	https://docs.filecoin.io/get-started/lotus/installation/#software-dependencies

	'''
	sudo apt update && sudo apt install mesa-opencl-icd ocl-icd-opencl-dev pkg-config build-essential libclang-dev gcc git bzr jq curl -y && sudo apt upgrade -y
	'''
	
5. ~~发现系统的apt 源还是默认源，有点慢，暂时不改了；~~ 没能忍，改成了清华的源重新做了一次（忘了留网址，网上搜索即可）；
6. 顺利make ，并make install，就是make的时间比较长大概有二十分钟？