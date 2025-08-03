1. 查看当前是否存在虚拟内存
```shell
sudo swapon -s
```
2. 创建swap文件
   - /swapfile 是文件名及地址，后续所有配置地址需要一致
```shell
# /swapfile  创建2G的swap文件
sudo fallocate -l 2G /swapfile
```

1. 设置文件权限
```shell
# 600
sudo chmod 600 /swapfile
```
1. 格式化swap文件
```shell
sudo mkswap /swapfile
```
1. 启用swap文件
```shell
sudo swapon /swapfile
```

1. 设置开机自动挂载swap文件
```shell
sudo echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

7.查看配置结果
```shell
sudo swapon -s
```