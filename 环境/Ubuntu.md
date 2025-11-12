# Ubuntu

## ufw

1. 查看状态

    ```bash
    sudo ufw status verbose
    ```

2. 启用防火墙

   ```bash
   # 启用前确保是否开启ssh访问端口
   sudo ufw enable
   ```

3. 更新防火墙版本

    ```bash
    sudo apt update
    ```

4. 安装防火墙

    ```bash
    sudo apt install ufw -y
    ```

5. 添加放行规则

    ```bash
    sudo ufw allow 80/tcp
    sudo ufw allow 443/tcp
    sudo ufw allow 8080/tcp

    # 放行源IP所有端口
    sudo ufw allow from 202.107.203.91

    # 放行指定IP的指定端口
    sudo ufw allow from 192.168.1.100 to any port 3306 proto tcp

    # 设置默认拒绝策略
    # 默认入站策略改成拒绝
    sudo ufw default deny incoming
    # 默认出站策略改成允许
    sudo ufw default allow outgoing
    ```

6. 重载防火墙配置

    ```bash
    sudo ufw reload
    ```

7. 查看端口

    ```bash
    sudo ufw status numbered
    ```

8. 删除端口

   ```bash
   # 删除第一条规则
   sudo ufw delete 1
   ```

9. 配置文件路径

    | 文件                      | 作用                                                      |
    | ----------------------- | ------------------------------------------------------- |
    | `/etc/ufw/ufw.conf`     | UFW 是否启用、日志等级等全局设置                                      |
    | `/etc/default/ufw`      | 默认策略（DEFAULT_INPUT_POLICY、DEFAULT_OUTPUT_POLICY、IPV6 等） |
    | `/etc/ufw/user.rules`   | 用户自定义规则（IPv4）                                           |
    | `/etc/ufw/user6.rules`  | 用户自定义规则（IPv6）                                           |
    | `/etc/ufw/before.rules` | UFW 启动前应用的底层 iptables 规则                                |
    | `/etc/ufw/after.rules`  | UFW 启动后应用的底层 iptables 规则                                |
