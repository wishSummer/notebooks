# 清理端口占用

1. 查看端口占用程序 PID

    ```shell
    netstat -ano | findstr ":80" | findstr "LISTENING"
    ```

2. 杀死进程

    ```shell
    taskkill -PID  <PID> -F
    ```
