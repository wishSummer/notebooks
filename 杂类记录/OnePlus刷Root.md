# OnePlus 刷机 Root

> 一加手机 音量上 + 电源建 : 开关机
> 一加手机 音量下 + 电源键 : 进入 `fastboot` 模式

## 开启开发者模式、OEM 解锁、USB 调试

- 设置 → 关于手机 → 版本号 → 连续点几次 → 开发者模式
- 设置 → 系统设置 → 开发者选项：
  - 开启 `OEM` 解锁
  - 开启 `USB` 调试

## 备份所有数据

## ADB 进入 `fastboot` → 解锁 `Bootloader`

- 电脑连接手机
- 管理员权限进入终端

```shell
# 查看是否获取到手机连接 会有手机id输出
adb devices

# 进入 fastboot 调试状态
adb reboot bootloader

# 开启 Bootloader，输入命令后 使用音量键选择选项，电源键确认
fastboot flashing unlock

# 查看 fastboot 状态 ： unlocked: yes (已解锁)
fastboot getvar unlocked

# 重启机器
fastboot reboot

```

## 获取当前手机的 `init_boot.img` 文件

- 参考教程 [Payload-Dumper-Compose 文档](https://magiskcn.com/payload-dumper-compose.html)

- 安装APP
  - 下载 `APP` `Payload-Dumper-Compose` [提取APP连接](https://github.com/rcmiku/Payload-Dumper-Compose/releases/tag/1.5)
  - 将 `APP` 传到手机安装。
- 在线提取 `init_boot.img`
  - 查看本机系统版本号
  - 查找与本机版本号相同的 rom，[资源路径](https://magiskcn.com/roms.html) **版本号必须要一致**
- 在手机端使用 `Payload-Dumper-Compose`
  - 点击链接
  - 输入找到的rom连接
  - 进行解析
  - 提取 init_boot.img 文件
- 备份 init_boot.img 文件到 `PC`，便于出错恢复手机。

## 用 `Magisk` 修补 `boot`

- 参考教程 [Payload-Dumper-Compose 文档](https://magiskcn.com/)

- 安装 APP
  - 下载 `APP` `Magisk` [提取APP连接](https://github.com/topjohnwu/Magisk/releases)
  - 将 `APP` 传到手机安装。
- 点击 安装 –> 选择 boot.img –> 开始修补文件 –> 修补完成

## `fastboot` 刷入修补镜像

- 将修补完成的 `magisk_patched-***` 文件剪切到 `PC` 端
- 在文件目录使用终端

```shell
# 手机进入 fastboot 模式
adb reboot bootloader

# 刷入镜像 如果 Payload-Dumper-Compose 提取的是 boot 使用
fastboot flash boot [面具修补生成文件]

# 刷入镜像 如果 Payload-Dumper-Compose 提取的是 init_boot 使用
fastboot flash init_boot [面具修补生成文件]

## 开机 → 完成 `Root`
fastboot reboot
```
