# Mysql连接池



    # 查看连接池连接
    SHOW PROCESSLIST;

    // 清理闲置连接
    select concat('KILL ',id,';') from information\_schema.processlist where Command = 'Sleep';
    kill (连接Id);

    // 查看链接
    select p.db,count(id) from information\_schema.processlist  p where Command = 'Sleep' GROUP BY db;

    # 查看最大连接数
    show global variables like '%max\_connect\_errors%';

    ```

