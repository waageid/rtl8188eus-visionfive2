# rtl8188eus

## 直接使用
> 不适用于 `2023-02` 版本的 `debian` 镜像

点击[Release](https://github.com/waageid/rtl8188eus-visionfive2/releases)下载`.ko`文件，直接加载内核模块

## 从源码编译
> 由于原仓库中 `Makefile` 文件在处理 `ARCH` 类型的时候会将 `risv` 架构解释成 `riscv64`, 导致编译失败；  
> 又由于目前 官方提供的[`debian` 镜像](https://drive.google.com/drive/folders/1cctIVdCfbPhKpyQ0PcmCQ92KCQjJ8JI5)(2023-03)`linux 内核`头文件不完整，导致直接在开发板上编译时出现内核头文件缺失的问题；  
> 以下流程在`2023-03 debain系统`下测试验证

## 文件目录结构
```
`-- repos
    |-- linux
    `-- rtl8188eus-visionfive2 
```

### 安装依赖
```sh
apt-get install libncurses-dev libssl-dev bc flex bison make gcc
# 如果交叉编译需要添加 `gcc-riscv64-linux-gnu`
```

### 编译`linux`内核

> 内核仓库: [https://github.com/starfive-tech/linux](https://github.com/starfive-tech/linux)

1. 拉取源码
```sh
git clone https://github.com/starfive-tech/linux.git --depth=1 -b JH7110_VisionFive2_devel
```

2. 配置及编译
```sh
cd linux

make starfive_visionfive2_defconfig
# 或交叉编译
# make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- starfive_visionfive2_defconfig

make menuconfig
# 或交叉编译
# make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- menuconfig

# 执行 make menuconfig 后，依次按 General setup -> Local version - append to kernel release，编辑添加后面括号中版本后缀，包含短横线(-starfive)，按Save保存为 .config 编译配置（这一步目前还是需要的，通过执行 uname -r，如输出 5.15.0-starfive，则这一步不可省略，省略后会导致后面编译的 .ko模块内核版本不匹配）

make -j4
# 或交叉编译
# make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -j4

# -j4指定编译的线程数
```

### 编译 rtl8188eus

1. 准备   
> 找到以下内容
```Makefile
all: modules

modules:
	$(MAKE) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) -C $(KSRC) M=$(shell pwd)  modules
```

> 将

```Makefile
$(MAKE) ARCH=$(ARCH) CROSS_COMPILE=$(CROSS_COMPILE) -C $(KSRC) M=$(shell pwd)  modules
```

> 修改为

```Makefile
make ARCH=riscv -C ../linux M=$(shell pwd) modules -j4
# ../linux 为内核源码路径，-j4 指定编译的线程数
# 或交叉编译（未验证）
# make ARCH=riscv CROSS_COMPILE=riscv64-linux-gnu- -C ../linux M=$(shell pwd) modules -j4
```
2. 编译
```sh
make
```

2. 安装
```shell
make install
```

3. 卸载
```sh
make uninstall
```