# Linux

## ll 和 ls

> 罗列出当前文件或目录的详细信息，含有时间、读写权限、大小、时间等信息 ，像Windows显示的详细信息。ll是“ls -l"的别名。`可以理解为 ll 和 ls -l 的功能是相同的。`

- -a 列出目录下的所有文件，包括以 . 开头的隐含文件。
- -A 显示除 “.”和“..”外的所有文件。
- -l 列出文件的详细信息。
- -s 在每个文件名后输出该文件的大小。`
- -t 以时间降序排序,默认升序。
- --color=no 不显示彩色文件名
  - 蓝色-->目录
  - 绿色-->可执行文件 
  - 红色-->压缩文件
  - 浅蓝色-->链接文件
  - 灰色-->其他文件
- -p -F 在每个文件名后附上一个字符以说明该文件的类型。
	- "*":表示可执行的普通文件；
	- "/":表示目录；
	- “@”:表示符号链接；
	- “|”:表示FIFOs；
	- “=”:表示套接字(sockets)。
- -S 以文件大小排序。
- -u 以文件上次被访问的时间排序。
- -m 横向输出文件名，并以“，”作分格符。
- -R 列出所有子目录下的文件。
- -X 以文件的扩展名(最后一个 . 后的字符)排序。
- -N 不限制文件长度。
- -b 把文件名中不可输出的字符用反斜杠加字符编号(就象在C语言里一样)的形式列出。
- -1 一行只输出一个文件。
- -c 输出文件的 i 节点的修改时间，并以此排序。
- -d 将目录像文件一样显示，而不是显示其下的文件。
- -i 输出文件的 i 节点的索引信息。
- -q 用?代替不可输出的字符。
- -r 对目录反向排序。
- -x 按列输出，横向排序。
- -B 不输出以 “~”结尾的备份文件。
- -C 按列输出，纵向排序。
- -G 输出文件的组的信息。
- -L 列出链接文件名而不是链接到的文件。
- -Q 把输出的文件名用双引号括起来。
- --help 在标准输出上显示帮助信息。
- --version 在标准输出上输出版本信息并退出。
## wc
> 用于计算文件的行数、字数和字节数。

- -l , --lines : 统计显示行数；
- -w , --words : 统计显示字数；
- -m , --chars : 统计显示字符数；
- -c , --bytes : 统计显示字节数；
- -L , --max-line-length : 显示最长行的长度；

## grep
> Linux grep (global regular expression) 命令用于查找文件里`符合条件的字符串或正则表达式。`
> grep 指令用于查找内容包含指定的范本样式的文件，如果发现某文件的内容符合所指定的范本样式，预设 grep 指令会把含有范本样式的那一列显示出来。若不指定任何文件名称，或是所给予的文件名为 -，则 grep 指令会从标准输入设备读取数据。

**常用命令**
```shell
# pattern - 表示要查找的字符串或正则表达式。
# files - 表示要查找的文件名，可以同时查找多个文件，如果省略 files 参数，则默认从标准输入中读取数据。
grep [options] pattern [files]
grep -n '2019-10-24 00:01:11' *.log

# 在当前目录里第一级文件夹中寻找包含指定字符串的 .in 文件
grep "thermcontact" /.in

# 在文件夹 dir 中递归查找所有文件中匹配正则表达式 "pattern" 的行，并打印匹配行所在的文件名和行号
grep -rn pattern dir/

# 在当前目录中，查找后缀有 file 字样的文件中包含 test 字符串的文件，并打印出该字符串的行。
grep test *file 

# 从当前目录开始查找所有扩展名为 .in 的文本文件，并找出包含 "thermcontact" 的行
# xargs 是一个强有力的命令，它能够捕获一个命令的输出，然后传递给另外一个命令。
find . -name "*.in" | xargs grep "thermcontact"

# 查找文件 统计行数。
find -type f |grep wc -l
```

- type 
  - d 指定查找目录
  - f 指定查找文件
- -i：忽略大小写进行匹配。
- -v：反向查找，只打印不匹配的行。
- -n：显示匹配行的行号。
- -r：递归查找子目录中的文件。
- -l：只打印匹配的文件名。
- -c：只打印匹配的行数。
- -A<显示行数> 或 --after-context=<显示行数> : 除了显示符合范本样式的那一列之外，并显示该行之后的内容。
- -B<显示行数> 或 --before-context=<显示行数> : 除了显示符合样式的那一行之外，并显示该行之前的内容。
- -C<显示行数> 或 --context=<显示行数>或-<显示行数> : 除了显示符合样式的那一行之外，并显示该行前后的内容。
- -a 或 --text : 不要忽略二进制的数据。
- -b 或 --byte-offset : 在显示符合样式的那一行之前，标示出该行第一个字符的编号。
- -e<范本样式> 或 --regexp=<范本样式> : 指定字符串做为查找文件内容的样式。
- -E 或 --extended-regexp : 将样式为延伸的正则表达式来使用。
- -f<规则文件> 或 --file=<规则文件> : 指定规则文件，其内容含有一个或多个规则样式，让grep查找符合规则条件的文件内容，格式为每行一个规则样式。
- -F 或 --fixed-regexp : 将样式视为固定字符串的列表。
- -G 或 --basic-regexp : 将样式视为普通的表示法来使用。
- -h 或 --no-filename : 在显示符合样式的那一行之前，不标示该行所属的文件名称。
- -H 或 --with-filename : 在显示符合样式的那一行之前，表示该行所属的文件名称。
- -l 或 --file-with-matches : 列出文件内容符合指定的样式的文件名称。
- -L 或 --files-without-match : 列出文件内容不符合指定的样式的文件名称。
- -o 或 --only-matching : 只显示匹配PATTERN 部分。
- -q 或 --quiet或--silent : 不显示任何信息。
- -s 或 --no-messages : 不显示错误信息。
- -V 或 --version : 显示版本信息。
- -w 或 --word-regexp : 只显示全字符合的列。
- -x --line-regexp : 只显示全列符合的列。


## rm
> 用于删除一个文件或者目录。

**常用命令**
```shell
# 删除目标目录。
rm -rf [directoryName]
```

- -f 即使原档案属性设为唯读，亦直接删除，无需逐一确认。
- -r 将目录及以下之档案亦逐一删除。
- -i 删除前逐一询问确认，默认不询问。

## mv

> 命令用来`为文件或目录改名`、或`将文件或目录移入其它位置`。
> 注意：批量移动当前目录下文件时，需要`先执行显示隐藏文件命令`，否则隐藏文件以及隐藏文件夹`不会被移动到新目录`。`英语点号开头的文件`会被作为隐藏文件处理，`英语点号开头的文件夹`也被作为隐藏文件夹处理。
> 例如：文件 **.a.txt**， 目录 **.tp5**。

**常用命令**
``` shell
mv [options] source dest
```

- **-b**: 当目标文件或目录存在时，在执行覆盖前，`会为其创建一个备份`。
- **-i**: 如果指定移动的源目录或文件与目标的目录或文件同名，则会先`询问是否覆盖旧文件`，输入 y 表示直接覆盖，输入 n 表示取消该操作。
- **-f**: 如果指定移动的源目录或文件与目标的目录或文件同名，`不会询问，直接覆盖旧文件`。
- **-n**: `不覆盖`任何已存在的文件或目录。
- **-u**：当源文件比目标文件`新`或者目标文件`不存在`时，才执行移动操作。

## scp
> Linux scp 命令用于 Linux 之间复制文件和目录。
> scp 是 secure copy 的缩写, scp 是 linux 系统下基于 ssh 登陆进行安全的远程文件拷贝命令。
> scp 是加密的，rcp 是不加密的，scp 是 rcp 的加强版。

**常用命令**
```shell
# 复制文件
scp [option] file_source file_target 
# 复制目录
scp [option] local_file remote_username@remote_ip:remote_folder
# 如果路径中有空格，则必须使用双反斜杠 \\ 并将整个路径用引号引起来转义字符：
scp myfile.txt user@192.168.1.100:"/file\\ path\\ with\\ spaces/myfile.txt"
```

- -r： 递归复制整个目录。
- -p：保留原文件的修改时间，访问时间和访问权限。
- -P port：注意是大写的P, port是指定数据传输用到的端口号
- -1： 强制scp命令使用协议ssh1
- -2： 强制scp命令使用协议ssh2
- -4： 强制scp命令只使用IPv4寻址
- -6： 强制scp命令只使用IPv6寻址
- -B： 使用批处理模式（传输过程中不询问传输口令或短语）
- -C： 允许压缩。（将-C标志传递给ssh，从而打开压缩功能）
- -q： 不显示传输进度条。
- -v：详细方式显示输出。scp和ssh(1)会显示出整个过程的调试信息。这些信息用于调试连接，验证和配置问题。
- -c cipher： 以cipher将数据传输进行加密，这个选项将直接传递给ssh。
- -F ssh_config： 指定一个替代的ssh配置文件，此参数直接传递给ssh。
- -i identity_file： 从指定文件中读取传输时使用的密钥文件，此参数直接传递给ssh。
- -l limit： 限定用户所能使用的带宽，以Kbit/s为单位。
- -o ssh_option： 如果习惯于使用ssh_config(5)中的参数传递方式，
- -S program： 指定加密传输时所使用的程序。此程序必须能够理解ssh(1)的选项。

## find
> Linux find 命令用于在指定目录下`查找文件和目录`，它可以使用不同的选项来过滤和限制查找的结果。

## chmod TODO

## chown TODO

## chgrp TODO

## tar TODO

## gzip TODO

## gunzip TODO

**常用命令**
``` shell
# 是要查找的`目录路径`，可以是一个`目录或文件名`，也可以是`多个路径`，多个路径之间用`空格分隔`，如果`未指定路径则默认为当前目录`。
# 动作: 可选的，用于对匹配到的文件执行操作，比如删除、复制等。
find [路径] [匹配条件] [动作]
# 找并执行操作（例如删除）：
# -exec 选项允许你执行一个命令，{} 将会被匹配到的文件名替代，\; 表示命令结束。
find /path/to/search -name "pattern" -exec rm {} \;

find . -type d ! \( -name '215fa58a59b3dd95334eedf04904274355703c5e4d8618b85266b0613ef0ff00' -o -name 'ebaffbcaa889184618f04162b39bafaba2ff556e94613288c14b7635df2a0d40' \) -exec rm -rf {} \;
```

**常规命令**
- `-name pattern`：按文件名查找，支持使用通配符 `*` 和 `?`。

- `-type type`：按文件类型查找，可以是 `f`（普通文件）、`d`（目录）、`l`（符号链接）等。

- `-size [+-]size[cwbkMG]`：按文件大小查找，支持使用 `+` 或 `-` 表示大于或小于指定大小，单位可以是 `c`（字节）、`w`（字数）、`b`（块数）、`k`（KB）、`M`（MB）或 `G`（GB）。

- `-mtime days`：按修改时间查找，支持使用 `+` 或 `-` 表示在指定天数前或后，days 是一个整数表示天数。

- `-user username`：按文件所有者查找。

- `-group groupname`：按文件所属组查找。


**find 命令中用于时间查找参数**

- `-amin n`：查找在 n 分钟内被访问过的文件。

- `-atime n`：查找在 n*24 小时内被访问过的文件。

- `-cmin n`：查找在 n 分钟内状态发生变化的文件（例如权限）。

- `-ctime n`：查找在 n*24 小时内状态发生变化的文件（例如权限）。

- `-mmin n`：查找在 n 分钟内被修改过的文件。

- `-mtime n`：查找在 n*24 小时内被修改过的文件。
  关于时间 n 参数的说明：
  - **+n**：查找 n 天前更早的文件或目录。
  - **-n**：查找在 n 天内更改过属性的文件或目录。
  - **n**：查找在 n 天前（指定那一天）更改过属性的文件或目录。
  - **0** : 是当前时间。

## mount
> 用于挂载Linux系统外的文件。Linux磁盘与Windows分区不太相同，需要将磁盘挂载到某一目录下，相当于Windows分区，才能够使用磁盘。磁盘挂载到某一目录后，源目录下文件会被隐藏，当unmonut（卸载）后，会自动恢复。

**常用命令**
```shell
# 将 /dev/hda1 用唯读模式挂在 /mnt 之下。
mount -o ro /dev/hda1 /mnt
```

- -V：显示程序版本
- -h：显示辅助讯息
- -v：显示较讯息，通常和 -f 用来除错。
- -a：将 /etc/fstab 中定义的所有档案系统挂上。
- -F：这个命令通常和 -a 一起使用，它会为每一个 mount 的动作产生一个行程负责执行。在系统需要挂上大量 NFS 档案系统时可以加快挂上的动作。
- -f：通常用在除错的用途。它会使 mount 并不执行实际挂上的动作，而是模拟整个挂上的过程。通常会和 -v 一起使用。
- -n：一般而言，mount 在挂上后会在 /etc/mtab 中写入一笔资料。但在系统中没有可写入档案系统存在的情况下可以用这个选项取消这个动作。
- -s-r：等于 -o ro
- -w：等于 -o rw
- -L：将含有特定标签的硬盘分割挂上。
- -U：将档案分割序号为 的档案系统挂下。-L 和 -U 必须在/proc/partition 这种档案存在时才有意义。
- -t：指定档案系统的型态，通常不必指定。mount 会自动选择正确的型态。
- -o async：打开非同步模式，所有的档案读写动作都会用非同步模式执行。
- -o sync：在同步模式下执行。
- -o atime、-o noatime：当 atime 打开时，系统会在每次读取档案时更新档案的『上一次调用时间』。当我们使用 flash 档案系统时可能会选项把这个选项关闭以减少写入的次数。
- -o auto、-o noauto：打开/关闭自动挂上模式。
- -o defaults:使用预设的选项 rw, suid, dev, exec, auto, nouser, and async.
- -o dev、-o nodev-o exec、-o noexec允许执行档被执行。
- -o suid、-o nosuid：
允许执行档在 root 权限下执行。
- -o user、-o nouser：使用者可以执行 mount/umount 的动作。
- -o remount：将一个已经挂下的档案系统重新用不同的方式挂上。例如原先是唯读的系统，现在用可读写的模式重新挂上。
- -o ro：用唯读模式挂上。
- -o rw：用可读写模式挂上。
- -o loop=：使用 loop 模式用来将一个档案当成硬盘分割挂上系统。

## du
> Linux du （英文全拼：disk usage）命令用于显示目录或文件的大小。
> du 会显示指定的目录或文件所占用的磁盘空间。

**常用命令**
```shell
# 查看文件或目录磁盘的空间使用情况
du -h --max-depth=1

# 查看目录下文件大小
du -sh *	 

# 查看文件或目录大小
du -sh (文件名、文件目录) 

```

- -h或--human-readable 以K，M，G为单位，提高信息的可读性。
- -H或--si 与-h参数相同，但是K，M，G是以1000为换算单位。
- --max-depth=<目录层数> 超过指定层数的目录后，予以忽略。
- -s或--summarize 仅显示指定目录或文件的总大小，而不显示其子目录的大小。
- -S或--separate-dirs 显示个别目录的大小时，并不含其子目录的大小。
- -a或-all 显示目录中个别文件的大小。
- -b或-bytes 显示目录或文件大小时，以byte为单位。
- -k或--kilobytes 以1024 bytes为单位。
- -c或--total 除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
- -D或--dereference-args 显示指定符号连接的源文件大小。
- -l或--count-links 重复计算硬件连接的文件。
- -L<符号连接>或--dereference<符号连接> 显示选项中所指定符号连接的源文件大小。
- -m或--megabytes 以1MB为单位。
- -x或--one-file-xystem 以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
- -X<文件>或--exclude-from=<文件> 在<文件>指定目录或文件。
- --exclude=<目录或文件> 略过指定的目录或文件。
- --help 显示帮助。
- --version 显示版本信息。

## df
> Linux df（英文全拼：disk free） 命令用于显示目前在 Linux 系统上的文件系统磁盘使用情况统计。

**常用命令**
```shell
# 显示磁盘分区上的可使用磁盘空间(根路径)
df -h

```

- -h, --human-readable 使用人类可读的格式
- -H, --si 很像 -h, 但是用 1000 为单位而不是用 1024
- -a, --all 包含所有的具有 0 Blocks 的文件系统
- --block-size={SIZE} 使用 {SIZE} 大小的 Blocks
- -i, --inodes 列出 inode 资讯，不列出已使用 block
- -k, --kilobytes 就像是 --block-size=1024
- -l, --local 限制列出的文件结构
- -m, --megabytes 就像 --block-size=1048576
- --no-sync 取得资讯前不 sync (预设值)
- -P, --portability 使用 POSIX 输出格式
- --sync 在取得资讯前 sync
- -t, --type=TYPE 限制列出文件系统的 TYPE
- -T, --print-type 显示文件系统的形式
- -x, --exclude-type=TYPE 限制列出文件系统不要显示 TYPE
- --help 显示这个帮手并且离开
- --version 输出版本资讯并且离开

## top TODO

## ping TODO

## ifconfig TODO

## ss
> Socket Statistics，ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。ss能够显示更多更详细的有关TCP和连接状态的信息，比netstat更快速更高效。
> 当服务器的socket连接数量变得非常大时，无论是使用netstat命令还是直接cat /proc/net/tcp，执行速度都会很慢。
ss利用到了TCP协议栈中tcp_diag。tcp_diag是一个用于分析统计的模块，可以获得Linux 内核中第一手的信息，这就确保了ss的快捷高效。

**常用命令**
```shell
# 查看主机监听的tcp端口
ss -tnl

# 使用 -p 选项查看监听端口的程序名称
ss -tlp

# -a –all 对 TCP 协议来说，既包含监听的端口，也包含建立的连接
$ ss -tna

# 获取ipv4 处于 listening状态的结果
ss -4 state listening

# 同时过滤 TCP 的状态和端口号，要有使用 \ 转义小括号
ss -4n state listening \( dport = :ssh \)

# 列举出处于 FIN-WAIT-1状态的源端口为 80 或者 443，目标网络为 193.233.7/24 所有 TCP 套接字：
$ ss state fin-wait-1 \( sport = :http or sport = :https \) dst 193.233.7/24 

# 匹配远程地址和端口号
$ ss dst 192.168.1.5
$ ss dst 192.168.119.113:http
$ ss dst 192.168.119.113:443

# 可以使用下面的语法做端口号的过滤 使用比较符需要转义
$ ss dport OP PORT
$ ss -tunl dport \< 50
```

- -s 展示各种协议的总连接数、侦听器数量和数据包统计
- -t, –tcp 显示 TCP 协议的 sockets
- -u, –udp 显示 UDP 协议的 sockets
- -x, –unix 显示 unix domain sockets，与 -f 选项相同
- -n, –numeric 不解析服务的名称，如 “22” 端口不会显示成 “ssh”
- -l, –listening 只显示处于监听状态的端口
- -p, –processes 显示监听端口的进程(Ubuntu 上需要 sudo)
- -a, –all 对 TCP 协议来说，既包含监听的端口，也包含建立的连接
- -r, –resolve 把 IP 解释为域名，把端口号解释为协议名称
- -o, –options 显示时间信息
- -m, –memory 显示 socket 使用的内存 
- -i, –info 显示更多 TCP 内部的信息
- -V, –version 显示版本号
- -h, –help 帮助
- 传输协议：

| 协议名称 | Option |
| :--: | :--: |
| PACKET |  |
| RAW |  |
| TCP | -t |
| UDP | -u |
| Unix Socket| -x |

- state
过滤TCP状态

| 状态名称 | 含义 |
| :--: | :--- |
| all | 查看所有TCP状态 |
| syn-sent | 客户端已发送连接请求（SYN），等待服务器确认 |
| syn-recv | 表示服务器已收到客户端连接请求（SYN），并已发送确认（SYN-ACK），等待客户端确认 |
|  established | 已建立连接，客户端和服务器可以进行数据传输 |
| fin-wait-1 | 表示TCP连接中的一端（通常是客户端）已发送连接终止请求（FIN），等待另一端确认或继续发送数据 |
| fin-wait-2 | 表示TCP连接中的一端已经收到了对方发送的连接终止请求（FIN），并且已经发送了确认，等待对方发送连接终止请求（FIN） |
| time-wait | 表示连接已经终止，但是为了确保所有的数据包都已经被完全传输，连接在此状态下会等待一段时间（两倍的最大报文段生存时间），然后再彻底关闭 |
| closing  | 表示TCP连接中的一端已经收到对方的连接终止请求（FIN），并且发送了确认，但是还未收到对方的确认 |
| closed | 表示连接已经完全关闭，不再传输数据 |
| close-wait | 表示对方（通常是服务器）已经发送了连接终止请求（FIN），等待本端（通常是客户端）发送确认或者继续发送数据 |
| last-ack | 表示对方（通常是客户端）已经确认了连接终止请求（FIN），并且已经发送了连接终止请求（FIN），等待本端（通常是服务器）确认 |
| listening | 表示服务器正在监听传入的连接请求 |
| connected | 已连接 |
| synchronized | 列出除了 syn-sent 之外的所有 TCP 状态 |
| bucket | 列出 maintained 的状态，如：time-wait 和 syn-recv |
| big | 列出和 bucket 相反的状态 |

- dst [ip:port]过滤连接目标
- src  [ip:port]过滤连接来源
- dport [op] [port] 源端口
- sport  [op] [port]目标端口
- 条件过滤端口 OP可替换为：

| 需转义字符 | 同义词 | 含义 |
| :--: | :--: | :--: |
| <= | le | 小于或等于某个端口号 |
| >= | ge | 大于或等于某个端口号 |
| == | eq | 等于某个端口号 |
| != | ne | 不等于某个端口号 |
| > | gt | 大于某个端口号 |
| < | lt | 小于某个端口号 |

## netstat
> 用于查看当前主机网络状态。

**常用命令**

```shell
netstat -anp|grep 8080
```

- -a或--all 显示所有连线中的Socket。
- -n或--numeric 直接使用IP地址，而不通过域名服务器。
- -p或--programs 显示正在使用Socket的程序识别码和程序名称。
- -s或--statistics 显示网络工作信息统计表。
- -l或--listening 显示监控中的服务器的Socket。
- -i或--interfaces 显示网络界面信息表单，显示网卡列表。
- -c或--continuous 持续列出网络状态。
- -t或--tcp 显示TCP传输协议的连线状况。
- -u或--udp 显示UDP传输协议的连线状况。
- -g或--groups 显示多重广播功能群组组员名单。
- -r或--route 显示Routing Table。
- -A<网络类型>或--<网络类型> 列出该网络类型连线中的相关地址。
- -C或--cache 显示路由器配置的快取信息。
- -e或--extend 显示网络其他相关信息。
- -F或--fib 显示路由缓存。
- -h或--help 在线帮助。
- -M或--masquerade 显示伪装的网络连线。
- -N或--netlink或--symbolic 显示网络硬件外围设备的符号连接名称。
- -o或--timers 显示计时器。
- -v或--verbose 显示指令执行过程。
- -V或--version 显示版本信息。
- -w或--raw 显示RAW传输协议的连线状况。
- -x或--unix 此参数的效果和指定"-A unix"参数相同。
- --ip或--inet 此参数的效果和指定"-A inet"参数相同。

## nmap
> 用于扫描目标主机信息
> - 查看网络实时信息
> - 查看网络上活动的所有 IP 的详细信息
> - 查看网络中打开的端口数量
> - 查看实时主机列表
> - 端口、操作系统和主机扫描

**常用命令**
```shell
# 扫描多个主机
nmap 103.76.228.244 157.240.198.35 172.217.27.174

# 扫描整个网段
nmap 103.76.228.*

#扫描 主机某个端口
nmap -p 443 80 22 10.0.0.1

#扫描主机一段连续端口
nmap -p 1-18 10.0.0.1

```

端口规格和扫描顺序：
- -v 展示更详细的信息
- -sA 探测firewall规则
- -sL 用于确认域名、Ip。
- -iL [需要扫描的文件] 可以导入文件路径，对文件内容（地址）进行扫描。
- -oG [存放文件]将扫描结果存入到指定文件。
- -p <端口范围>： 仅扫描指定端口
- -F：快速模式 - 扫描端口数少于指定的端口数。
- -sS 用于扫描主机开放端口
- -O 查看目标主机运行的操作系统，但不能获取精确系统版本


## ssh
> 加密的远程连接方式

**常用命令**
```shell
# 测试端口是否可连接
ssh -p [端口] [用户名]@[ip]
ssh -v -p 8080 root@192.168.1.154

```

- -v 会显示 SSH 客户端与服务器之间的详细通信过程，包括连接建立、密钥交换、认证过程等。

## telnet
用于远程连接或探测端口可用。
```shell
telnet 192.168.31.100 8080
```

## iptables
能够对网络数据包进出设备及转发进行控制。

内核会按照顺序依次检查 iptables 防火墙规则，如果发现**有匹配的规则目录**，则立刻执行相关动作，**停止继续向下查找规则目录**；如果所有的防火墙规则都**未能匹配成功**，则按照**默认策略处理**。使用**"-A"** 选项添加防火墙规则会将该规则**追加到整个链的最后**，而使用 **"-I"** 选项添加的防火墙规则则会**默认插入到链中作为第一条规则**。

**常用命令**
```shell
# 对规则的查看需要使用如下命令：
iptables -nvL
```

**iptables 默认维护 filter、nat、mangle、raw四张表以及INPUT、OUTPUT、FORWARD、PREROUTING、POSTROUTING内建链。**

- - Filter(过滤规则表)可控制链：
 	- INPUT（入站数据过滤）
	- OUTPUT（出站数据过滤）
	- FORWARD（转发数据过滤）
- Nat（地址转换规则表）
	- INPUT（入站数据过滤）
	- OUTPUT（出站数据过滤）
	- PREROUTING（路由前过滤）
	- POSTROUTING（路由后过滤）
- Mangle（修改数据标志位规则表，指定如何处理数据包。它能改变TCP头中的QoS位。）
	- INPUT（入站数据过滤）
	- OUTPUT（出站数据过滤）
	- FORWARM（转发数据过滤）
	- PREROUTING（路由前过滤）
	- POSTROUTING（路由后过滤）
- Row（控制Nat表中连接追踪机制的启用状况）
	- PREROUTING（路由前过滤）
	
	- OUTPUT（出站数据过滤）
  ![image-20240411203401567](/assets/image-20240411203401567.png "数据请求流程图")
> - 外部主机发送数据包给防火墙本机，数据将会经过 PREROUTING 链与 INPUT 链;
> - 如果是防火墙本机发送数据包到外部主机，数据将会经过 OUTPUT 链与 POSTROUTING 链;
> - 如果防火墙作为路由负责转发数据，则数据将经过 PREROUTING 链、FORWARD 链以及 POSTROUTING 链;
> 
```shell
# 命令基本格式
iptables [-t table] COMMAND [chain] CRETIRIA -j ACTION

# 添加规则
iptables -A INPUT -s 192.168.1.5 -j DROP

# 修改规则 修改第六行规则
iptables -R INPUT 6 -s 194.168.1.5 -j ACCEPT

# 删除规则 -s 194.168.1.5 -j ACCEPT可省略
iptables -D INPUT 6 -s 194.168.1.5 -j ACCEPT可省略

# 备份iptables规则
iptables-save > /etc/sysconfig/iptables

# 查看nat表
iptables-save -t nat

# 导入iptables规则
iptables-restore < 文件名称

# 添加iptables路由规则
iptables -t nat -A DOCKER-INGRESS 1 -p tcp -m tcp --dport 10001 -j DNAT --to-destination 172.19.0.2:10001

```

- 各参数含义
    - -t：指定需要维护的防火墙规则表 filter、nat、mangle或raw。在不使用 -t 时则默认使用 filter 表。
    - COMMAND：子命令，定义对规则的管理。
    - chain：指明链表。
    - CRETIRIA：匹配参数。
    - ACTION：触发动作。
- iptables 命令常用的选项及各自的功能：
	- -A  添加防火墙规则(添加到规则末尾)
	- -I  插入防火墙规则(插入到指定位置，未指定为首部)
	- -D  删除防火墙规则
	- -F  清空防火墙规则
	- -L  列出添加防火墙规则
	- -R  替换防火墙规则
	- -Z  清空防火墙数据表统计信息
	- -P  设置链默认规则
- iptables 命令常用匹配参数及各自的功能：
	- -n  表示不对 IP 地址进行反查，加上这个参数显示速度将会加快。 
	- -v 表示输出详细信息，包含通过该规则的数据包数量、总字节数以及相应的网络接口。
	- -p  匹配协议，! 表示取反
	- -s  匹配源地址
	- -d  匹配目标地址
	- -i  匹配入站网卡接口
	- -o  匹配出站网卡接口
	- --sport  匹配源端口
	- --dport  匹配目标端口
	- --src-range  匹配源地址范围
	- --dst-range  匹配目标地址范围
	- --limit  四配数据表速率
	- --mac-source  匹配源MAC地址
	- --sports  匹配源端口
	- --dports  匹配目标端口
	- --stste  匹配状态（INVALID、ESTABLISHED、NEW、RELATED)
	- --string  匹配应用层字串
	- --line-number 显示规则行号
- iptables 命令触发动作及各自的功能：
    - ACCEPT	 允许数据包通过
    - DROP		丢弃数据包
    - REJECT	   拒绝数据包通过
    - LOG		将数据包信息记录 syslog 曰志
    - DNAT 		 目标地址转换
    - SNAT 		 源地址转换
    - MASQUERADE  地址欺骗
    - REDIRECT	 重定向
## firewalld
>用于实时配置和管理系统的防火墙规则，并且不需要重新启动防火墙守护线程或服务。底层通过iptables、ip6tables、ebtables和nftables作为后端来处理网络数据包。
>不同于iptables基于规则先后顺序对数据包进行匹配处理;firewalld基于服务（service），区域（zone）概念将传入的流量分类到由源`IP、"/"、网络接口定义的区域`中，根据每个区域的配置处理数据包。

**常用命令**
```shell
# 设置默认zone
firewall-cmd --get-default-zone public 

# 查看活动区域和分配给它们的网络接口
firewall-cmd --get-active-zones 

# 打开防火墙
systemctl start firewalld
    
# 启用防火墙
systemctl enable firewalld

# 查看已经开放的端口
firewall-cmd --list-ports

# 开启public zone、 tcp、 22端口 、永久生效
firewall-cmd --zone=public --add-port=22/tcp --permanent
 # 指定 zone
  --zone=<zone>
 # 端口id / 协议
  --add-port=<portid>]/<protocol>
 # 永久开启，不添加则重启失效
  --permanent
    
# 关闭public zone、 tcp、 22端口 、永久生效
firewall-cmd --zone=public --remove-port=22/tcp --permanent

# 允许目标地址所有访问流量
firewall-cmd --zone=public --add-source=192.168.172.32 
    
# 重新加载防火墙 *重新加载过后才能生效
systemctl reload firewalld

```
**区域(zone)**
- block     拒绝外部网络连接。仅支持系统内部网络连接。
- dmz       仅允许选定的传入端口。
- drop      丢弃所有传入网络连接，并且仅允许传出网络连接。
- external  用于路由器连接类型。需要使用LAN和WAN接口，用以伪装（NAT）正常工作。
- home      仅允许选择的TCP / IP端口。
- internal  允许LAN上其他服务器或计算机，用于内部网络。
- public    允许所需的端口和服务。对于云服务器或您所托管的服务器使用类型。（默认）
- trust     接受所有网络连接。
- work      允许信任的用户和服务器访问。

**服务(service)**
> 服务是由本地、协议、源端口、目的地和防火墙助手模块组成的集合。
```shell
# 查看防火墙支持的服务类型
firewall-cmd --get-services

# 查看当前主机开启了那些服务
firewall-cmd --list-service

# 添加服务到 public zone
firewall-cmd --zone=public --add-service=dns --permanent 

# 删除服务
firewall-cmd --remove-service=cockpit --permanent 

```

## ps
> 命令用于显示当前进程的状态，类似于 windows 的任务管理器。

**常用命令**
```shell
# 查看指定进程信息
 ps -ef |grep java 

 # 显示所有进程的详细状态。
 ps -aux


```

- -A 列出所有的进程
- -w 显示加宽可以显示较多的资讯
- -u 查看指定用户信息
- -T 显示当前线程的层次结构。
- -aux 显示所有包含其他使用者的进程
输出格式 : USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND
  - USER: 行程拥有者
  - PID: pid
  - PPID：父进程ID，表示创建该进程的父进程的ID。
  - %CPU: 占用的 CPU 使用率
  - %MEM: 占用的内存使用率
  - VSZ: 占用的虚拟内存大小
  - RSS: 固定占用的内存大小
  - TTY: 终端类型，如果进程与某个终端关联，则显示该终端的名称；否则显示"?"。
  - STAT: 该行程的状态:
    - D: 无法中断的休眠状态 (通常 IO 的进程)
    - R: 正在执行中
    - S: 静止状态
    - T: 暂停执行
    - Z: 不存在但暂时无法消除
    - W: 没有足够的记忆体分页可分配
    - <: 高优先序的行程
    - N: 低优先序的行程
    - L: 有记忆体分页分配并锁在记忆体内 (实时系统或捱A I/O)
    - START: 行程开始时间
    - TIME: 执行的时间
    - COMMAND:所执行的指令
    

## nohup
> nohup 英文全称 no hang up（不挂起），用于后台运行程序，退出终端不会影响程序的运行。
> nohup 命令，在默认情况下（非重定向时），会输出一个名叫 nohup.out 的文件到当前目录下，如果当前目录的 nohup.out 文件不可写，输出重定向到 $HOME/nohup.out 文件中。

**常用命令**
```shell
# 语法格式
nohup Command [ Arg … ] [　& ]
nohup java -jar XXXX.jar &

# 停止程序
kill -9  进程号PID

# 在后台执行 root 目录下的 runoob.sh 脚本，并重定向输入到 runoob.log 文件：
# 2>&1 解释：
# 将标准错误 2 重定向到标准输出 &1 ，标准输出 &1 再被重定向输入到 runoob.log 文件中。
# 0 – stdin (standard input，标准输入)
# 1 – stdout (standard output，标准输出)
# 2 – stderr (standard error，标准错误输出)
nohup /root/runoob.sh > runoob.log 2>&1 &

```
如果后台运行成功终端页面会输出`appending output to nohup.out`，这时在root目录可以看到生成的nohup.out文件。
如果需要停止运行可以使用`ps -aux|grep [目标程序]` 查找nohup 运行脚本到 PID，然后使用 kill 命令来删除；

## kill TODO