## 说明

 30 年前Richard Stevens 在经典书《[**TCP**/IP**卷**](https://book.douban.com/subject/1088054/)》，这本书 2000 年翻译成中文版，豆瓣评分 9.2， 书中 Stevens 用了一个叫 sock 的程序来演示书中的各个理论知识点，这段代码我找到了，并且发现还能 work，将其在 aarch64 和 x86 平台均编译成功，分享给大家，同时缅怀一下已经过世的 Stevens

Wireshark 里有个菜单就是用 Stevens 来命名的：Statistics->TCP Stream Graphs->Time Sequence(Stevens)

## sock 编译

因为程序过于久远，基本无法编译通过，按照下面的步骤在 x86/aarch64 都编译成功

1. 安装 automake/autoconf

2. 修改configure ，去掉版本限制：

   ```
   ACLOCAL=${ACLOCAL-"${am_missing_run}aclocal-${am__api_version}"}
   
   AUTOMAKE=${AUTOMAKE-"${am_missing_run}automake-${am__api_version}"}  //去掉这个版本限制
   ```

3. configure 完后将 configure.in 重命名为configure.ac ，然后执行make

就会在 ./src/ 下看到一个可执行文件：sock



## 使用实例

```
// -O 9000 端口listen 后过 9 秒再 accept，这样有机会看到三次握手完成，但是在服务端全连接队里被填满
// -T SO_REUSEPORT 可以重用 time_wait/FIN_WAIT2 端口
// -F fork 支持多个客户端，这样 -q 设置全连接队列溢出才有意义
// -K 启用 keepalive ，但是默认 net.ipv4.tcp_keepalive_time 为 7200，需要自己改小后快速触发 keepalive 包
// -q 全连接队列，默认是 5，故意设为 1 可以更快地看到全连接队列溢出
// -P n  #ms to pause before first read or write (source/sink)，故意让数据在 buffer 里不读走，通过 netstat 可以看到数据，模拟应用繁忙
// -v debug 模式，输出更多信息
./src/sock -q 1 -F -v -T -O 9000 -P 5000 -Q 3000 -i  -s 0.0.0.0 4321
```

最佳学习方式：

最下面两个窗口左右分别是 tcpdump 和 netstat/ss 命令用来查看状态

![image-20240919154758343](https://cdn.jsdelivr.net/gh/plantegg/plantegg.github.io/images/951413iMgBlog/image-20240919154758343.png)

## 完整功能

```
#sock
usage: sock [ options ] <host> <port>              (for client; default)
       sock [ options ] -s [ <IPaddr> ] <port>     (for server)
       sock [ options ] -i <host> <port>           (for "source" client)
       sock [ options ] -i -s [ <IPaddr> ] <port>  (for "sink" server)
options: -b n  bind n as client's local port number
         -c    convert newline to CR/LF & vice versa
         -f a.b.c.d.p  foreign IP address = a.b.c.d, foreign port# = p
         -g a.b.c.d  loose source route
         -h    issue TCP half close on standard input EOF
         -i    "source" data to socket, "sink" data from socket (w/-s)
         -j a.b.c.d  join multicast group
         -k    write or writev in chunks
         -l a.b.c.d.p  client's local IP address = a.b.c.d, local port# = p
         -n n  #buffers to write for "source" client (default 1024)
         -o    do NOT connect UDP client
         -p n  #ms to pause before each read or write (source/sink)
         -q n  size of listen queue for TCP server (default 5)
         -r n  #bytes per read() for "sink" server (default 1024)
         -s    operate as server instead of client
         -t n  set multicast ttl
         -u    use UDP instead of TCP
         -v    verbose
         -w n  #bytes per write() for "source" client (default 1024)
         -x n  #ms for SO_RCVTIMEO (receive timeout)
         -y n  #ms for SO_SNDTIMEO (send timeout)
         -A    SO_REUSEADDR option
         -B    SO_BROADCAST option
         -C    set terminal to cbreak mode
         -D    SO_DEBUG option
         -E    IP_RECVDSTADDR option
         -F    fork after connection accepted (TCP concurrent server)
         -G a.b.c.d  strict source route
         -H n  IP_TOS option (16=min del, 8=max thru, 4=max rel, 2=min$)
         -I    SIGIO signal
         -J n  IP_TTL option
         -K    SO_KEEPALIVE option
         -L n  SO_LINGER option, n = linger time
         -N    TCP_NODELAY option
         -O n  #ms to pause after listen, but before first accept
         -P n  #ms to pause before first read or write (source/sink)
         -Q n  #ms to pause after receiving FIN, but before close
         -R n  SO_RCVBUF option
         -S n  SO_SNDBUF option
         -T    SO_REUSEPORT option
         -U n  enter urgent mode before write number n (source only)
         -V    use writev() instead of write(); enables -k too
         -W    ignore write errors for sink client
         -X n  TCP_MAXSEG option (set MSS)
         -Y    SO_DONTROUTE option
         -Z    MSG_PEEK
```



## 下载

https://www.icir.org/christian/sock.html 对应下载地址：https://www.icir.org/christian/downloads/sock-0.3.2.tar.gz


