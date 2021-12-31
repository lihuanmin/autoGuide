# /dev/null

## 概述

/dev/null(空设备)，是一个特殊的设备文件，它丢弃一切写入其中的数据，但是报告写入成功，读取它则会立即得到一个EOF。通常也被称为位桶或空洞。

## 用法

### 禁止标准输出

```shell
touch filename
echo hello > filename
cat filename 
# 界面展示 hello
cat filename > /dev/null
# 无任何展示
```

### 禁止错误输出

```shell
cat xxx
# cat: xxx: No such file or directory
cat xxx 2>/dev/nul
# 无任何展示
```

### 清空文件

```shell
echo hrro > hello 
cat hello
# hrro
cat /dev/null > hello 
cat hello 
# 无任何展示
```



# /dev/zero

## 概述

/dev/zero是一个特殊的设备，当你读取它的时候，他会提供无限多的空字符(NULL, ASCII(NUL), 0x00)

## 用法

### 生成指定大小的文件

```shell
dd if=/dev/zero of=file bs=1M count=1
# 生成一个1M大小的空文件
```

