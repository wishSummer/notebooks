# 使用 `alternatives` 管理多版本 `java jdk`

* 下载 jdk
> 清华源: https://mirrors.tuna.tsinghua.edu.cn/Adoptium/17/jdk/x64/linux/

```shell
# 解压压缩包
tar -zxvf jdk-17_linux-x64_bin.tar.gz
```

* 将jdk 注册到`alternatives`
```shell
# 创建JDK目录的软链接，并交由alternatives管理 
#                                链接       名称    路径                  优先级  
update-alternatives --install /usr/bin/java java /usr/java/jdk1.8.0_202/ 8

```