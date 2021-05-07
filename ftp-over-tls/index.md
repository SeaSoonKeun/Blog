# FTP over TLS

# FTP over TLS
FTP是互联网上广泛使用的文件传输协议，缺点是数据明文传输，在数据经过的节点上进行监听可轻易获取用户和密码，给安装带来诸多隐患，可使用SSL加密FTP连接。

先测试明文传输下用Tcpdump监听ftp用户名和密码：

 

```bash
tcpdump port 21 -nA

20:03:43.077038 IP 192.168.1.1.34453 > 192.168.1.4.21: Flags [P.], seq 21:34, ack 134, win 2920, length 13

E..5."@.@..J............k..3..(kP..hC...USER myhack58

20:03:43.077506 IP 192.168.1.4.21 > 192.168.1.1.34453: Flags [P.], seq 134:168, ack 34, win 365, length 34

E..J.g@.@.................(kk..@P..m#...331 Please specify the password.

20:03:43.081218 IP 192.168.1.1.34453 > 192.168.1.4.21: Flags [P.], seq 34:47, ack 168, win 2920, length 13

E..5.#@.@..I............k..@..(.P..h:...PASS myhack58

20:03:43.102350 IP 192.168.1.4.21 > 192.168.1.1.34453: Flags [P.], seq 168:191, ack 47, win 365, length 23

E..?.h@.@.................(.k..MP..m.8..230 Login successful.

20:03:43.103626 IP 192.168.1.1.34453 > 192.168.1.4.21: Flags [P.], seq 47:52, ack 191, win 2920, length 5

E..-.$@.@..P............k..M..(.P..h.L..PWD

20:03:43.104025 IP 192.168.1.4.21 > 192.168.1.1.34453: Flags [P.], seq 191:211, ack 52, win 365, length 20

E..<.i@.@.................(.k..RP..mt...257 "/home/myhack58"
```

 

查看Vsftpd是否支持SSL：

\#http://blog.myhack58.com

```bash
ldd `which vsftpd`|grep ssl

libssl.so.0.9.8 => /usr/lib/libssl.so.0.9.8 (0x00007f18f8c0a000)
```

 

生成SSL证书：

 

```bash
openssl req -new -x509 -nodes -out vsftpd.pem -keyout vsftpd.pem

chmod 400 vsftpd.pem

cp vsftpd.pem /etc/ssl/certs/
```

 

Vsftpd配置SSL支持：

 

```bash
ssl_enable=YES

allow_anon_ssl=YES

force_local_data_ssl=YES

force_local_logins_ssl=YES

force_anon_logins_ssl=YES

force_anon_data_ssl=YES

ssl_tlsv1=YES

ssl_sslv2=NO

ssl_sslv3=NO

rsa_cert_file=/etc/ssl/certs/vsftpd.pem
```

 

重启vsftp使用支持ssl的ftp客户端连接，本例使用FlashFXP连接：

![img](https://img-blog.csdn.net/20131113141959156?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbmV3Ym9ybjIwMTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
