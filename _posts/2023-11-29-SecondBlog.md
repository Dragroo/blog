---
layout: post
author: Dragroo
title: 使用tar压缩和解压缩文件
---
### 使用tar压缩和解压缩文件
1. 使用tar压缩文件

```bash
tar -zcvf test.tar.gz ./test/
```

该命令表示压缩当前文件夹下的文件夹test，压缩后缀名为test.tar.gz

如果不需要压缩成gz，只需要后缀为tar格式的，那么输入如下命令：

```bash
tar -cvf test.tar ./test/
```

2. 使用tar解压文件

```bash
tar -xzvf test.tar.gz  
```
该命令表示把后缀为.tar.gz的文件解压到当前文件夹下。

如果压缩文件的后缀是.tar，没有gz，则使用命令:
```bash
tar -xvf test.tar
```
