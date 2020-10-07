1. 分区：
使用parted 给sdc添加一个大于2T的分区
```
parted /dev/sdc
mklabel gpt
mkpart
	ext4
	0%
	100%
quit
```




Start         End     Sectors  Size
 2048 15628053134 15628051087  7.3T