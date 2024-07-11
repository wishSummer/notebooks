# 一、下载安装包

```shell
# 下载必要的构建工具
sudo yum install gcc openssl-devel bzip2-devel libffi-devel

# 下载安装包
wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz
wget https://www.python.org/ftp/python/3.9.1/Python-3.9.1.tgz
# 国内源
wget https://mirrors.huaweicloud.com/python/3.7.9/Python-3.7.9.tgz
```

# 二、安装

```shell
# 解压并进入 Python 3.7 目录
tar -zxvf Python-3.7.9.tgz
cd Python-3.7.9

# 使用--enable-optimizations选项编译Python解释器可以提高Python程序的执行速度，但可能会增加编译时间和生成的二进制文件的大小。
./configure --enable-optimizations

 # 根据你的 CPU 核心数调整 -j 参数
make -j 2 

# 使用make altinstall命令安装Python，会在系统中创建一个新的Python可执行文件，命名为pythonX.Y（其中X.Y表示Python的主版本和次版本号）。
 
```

