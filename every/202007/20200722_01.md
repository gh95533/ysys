[TOC]

# linux 文件基本操作管理



### 复制

在相同目录下复制，需要重新为目标文件起名

```
 cp gh.txt gh2.txt
```

在不同目录下，只需要跟着需要放置的路径

```
 cp gh2.txt ./filedir/
```

将文件夹下的内容拷贝到其他文件夹下

```
cp -r filedir/* ./filedir2/
```

### 移动

move

### 创建文件夹

mkdir  filedir

mkdir -p filedir/filedir2/filedir3

### 删除

rm -rf