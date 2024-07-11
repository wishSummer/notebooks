# centOS

## 一、安装Python

*   查看linux版本

    ```shell
    uname -a
    ```
*   当前环境未ubantu，默认安装有Python3

    ```shell
    # 查看当前主机python版本
    python
    ```

## 二、安装wget

*   centOS

    ```shell
    yum install wget
    ```

## 三、安装Selenium

    ```shell
    # 如果没有提示不识别pip，进行下载
    yum -y install python-pip

    pip install selenium
    ```

## 四、安装google-chrome

```shell
# 下载chrome安装包
wget https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm

# 安装 localinstall会自动下载相关依赖
yum localinstall google-chrome-stable_current_x86_64.rpm

# 查看当前chrome版本
google-chrome --version
```

## 五、下载ChromeDriver

*   最新版本查询链接
    <https://getwebdriver.com/chromedriver#stable>
*   旧版本查询链接
    <http://chromedriver.storage.googleapis.com/index.html>
*   配置chromeDriver

```shell
# 下载
wget https://storage.googleapis.com/chrome-for-testing-public/124.0.6367.201/linux64/chromedriver-linux64.zip

# 下载解压工具
yum install zip unzip

# 解压压缩包
unzip 文件名.zip

```

# ubantu

## 一、安装Python

*   查看linux版本
    ```shell
    uname -a
    ```
*   当前环境未ubantu，默认安装有Python3

```shell
# 查看当前主机python版本
python
```

## 二、安装wget

*   ubantu
    ```shell
    sudo apt-get update
    sudo apt-get install wget
    # 查看version
    wget
    ```

## 三、安装Selenium

```shell
```

## 四、安装google-chrome

```shell
# 下载chrome安装包
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb

# 安装
apt-get install ./google-chrome-stable_current_amd64.deb

```

