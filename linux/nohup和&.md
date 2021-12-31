# 执行文件

## 创建文件

创建一个shell脚本, test.sh

```shell
#!/bin/bash
i=0
while ((1))
do
 echo "Hello, $i"
 let i++
 sleep 1s
done
```

## 在终端执行

![](./pic/origin_sh.png)

## `Ctrl+c中断`

sh直接执行，`ctrl + c`文件执行**中断**

![](./pic/origin_sh_ctrl_c.png)

## 关闭session

关闭session文件执行**中断**

# $

## 创建文件

创建一个shell脚本，test.sh

```shell
#!/bin/bash
i=0
while ((1))
do
 echo "Hello, $i"
 let i++
 sleep 1s
done
```

## 在终端执行文件

![](./pic/execute_file_&_origin.png)

## `ctrl+c`中断

![](./pic/&_excute_file.png)

`ctrl+c`中断过后，程序**不中断**。

## 关闭session

关闭session，进程**中断**。

# nohup

## 创建文件

创建一个shell脚本，test.sh

```shell
#!/bin/bash
i=0
while ((1))
do
 echo "Hello, $i"
 let i++
 sleep 1s
done
```

## 在终端执行

![](./pic/nohup_origin_excute_file.png)

## `ctrl+c`中断

程序**中断**

## 关闭session

程序**不中断**

# 总结

| 操作        | 原始的 | &      | nohup  |
| ----------- | ------ | ------ | ------ |
| `ctrl+c`    | 中断   | 不中断 | 中断   |
| 关闭session | 中断   | 中断   | 不中断 |





