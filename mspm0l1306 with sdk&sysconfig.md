
## 软件安装

首先需要安装 [SDK](https://www.ti.com/tool/MSPM0-SDK) 和 [SYSCONFIG](https://www.ti.com/tool/SYSCONFIG)

![image.png](https://s2.loli.net/2024/05/01/WBSRyHGcojzECZD.png)
![image.png](https://s2.loli.net/2024/05/01/hx8qmwv2a54tU1p.png)

## 配置生成

打开 `SYSCONFIG` 软件产品选择 `SDK` 的安装路径, 选择对应芯片型号,点击 `START`

![image.png](https://s2.loli.net/2024/05/01/8pR4A7kX6GSUf9i.png)

这里我添加了一个 `GPIO` 

![image.png](https://s2.loli.net/2024/05/01/vZdy1ePNsgtTn2E.png)

这里生成9个文件

![image.png](https://s2.loli.net/2024/05/01/FXZxROENwf5nvty.png)

主要就是 `ti_msp_dl_config.h` `ti_msp_dl_config.c` 文件, 将这两文件复制到项目目录下的 `sysconfig` 目录

`ti_msp_dl_config.h`

![image.png](https://s2.loli.net/2024/05/01/A8w9QS4VeXTtqiv.png)

对于初始化设备， 不用我们担心, 类似 `CubeMX` 不用我们手动写 `Init` 函数。

## 编译

`make`

![image.png](https://s2.loli.net/2024/05/01/I9jgrD4GZoPfN1W.png)

