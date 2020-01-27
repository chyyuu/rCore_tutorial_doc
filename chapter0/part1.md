

下面的实验环境建立方式由简单到相对复杂一些，同学们可以基于自己的情况选择合适的实验方式。

## 在线环境下运行实验  

目前在线实验环境是[基于实验楼的在线实验环境](https://www.shiyanlou.com/courses/1481)。用户只需有一个能够上网的browser即可进行实验。首先需要在[实验楼](https://www.shiyanlou.com/)上注册一个账号，然后在[rcore在线实验环境](https://www.shiyanlou.com/courses/1481)的网页上输入验证码：wfkblCQp 就可以进入在线的实验环境。尝试执行下面的命令就开始进行实验了。

```shell
#编译
cd rCore_tutorial;  git checkout master; make all
#运行
make run
```



## docker环境下运行实验  

我们支持docker环境下进行实现，在docker hub上已有可用的docker环境，在当前目录下运行`make docker`将会从云端拉取docker镜像，并将当前目录挂载到/mnt位置。

```shell	
# 启动docker环境
make docker # 会进入docker中的终端
cd /mnt
# 然后可以进行编译/qemu中运行实验。例如：
# 编译用户态app组成的image
cd usr
make user_img
# 编译内核
cd ../os
make build
# 运行
make run
```

如有兴趣，也可以自行构建/调整docker镜像，相关的Dockerfile文件在当前目录下，我们提供了`make docker_build`命令来帮助构建，详情请看Dockerfile和Makefile

## 本地Linux环境下运行实验  

我们也支持本地Linux环境下开展实验，不过需要提前安装相关软件包，如rustc nightly，qemu-4.1.0+，device-tree-compiler 等。具体细节可参考[支持docker建立的Dockerfile](https://github.com/rcore-os/rCore_tutorial/blob/master/Dockerfile)和[支持github自动测试的main.yml](https://github.com/rcore-os/rCore_tutorial/blob/master/.github/workflows/main.yml)。假定安装好了相关软件，直接只需下面的命令，即可进行实验：

```shell
#在把实验代码下载到本地
git clone  https://github.com/rcore-os/rCore_tutorial.git
#编译
cd rCore_tutorial;  git checkout master; make all
#运行
make run
#如果一切正常，则qemu模拟的risc-v64计算机将输出

OpenSBI v0.4 (Jul  2 2019 11:53:53)
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name          : QEMU Virt Machine
Platform HART Features : RV64ACDFIMSU
Platform Max HARTs     : 8
Current Hart           : 0
Firmware Base          : 0x80000000
Firmware Size          : 112 KB
Runtime SBI Version    : 0.1

PMP0: 0x0000000080000000-0x000000008001ffff (A)
PMP1: 0x0000000000000000-0xffffffffffffffff (A,R,W,X)
switch satp from 0x8000000000080255 to 0x800000000008100e
++++ setup memory!    ++++
++++ setup interrupt! ++++
available programs in rust/ are:
  .
  ..
  user_shell
  notebook
  hello_world
  model
++++ setup fs!        ++++
++++ setup process!   ++++
++++ setup timer!     ++++
Rust user shell
>> 
```