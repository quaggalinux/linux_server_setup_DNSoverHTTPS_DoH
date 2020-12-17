# linux_server_setup_DNSoverHTTPS_DoH
linux服务器设置DNS查询为DNS over HTTPS(DoH)方式  
   
DNS over HTTPS(DoH)作为一种新的DNS保密查询标准,在浏览器是比较方便设置的,但是现阶段作为一台linux服务器想把底层的整个DNS查询切换成纯DNS over HTTPS(DoH)方式是没有标准做法的,因为现在所有linux发行版的各种网络管理进程都没有内置DoH的设置选项  
  
那有没有必要把整台linux服务器的DNS查询切换成纯DNS over HTTPS(DoH)方式呢?这个问题是见仁见智的,我们先不管有没有必要,而是考虑有没有一个简单的方法实现这个配置操作  
 
下面是我总结的其中一种认为简单的配置流程,这个流程先要用到CloudFlare的一个DoH守候进程程序,这个程序可以在下面的CloudFlare网站连接下载,里面列出有各种平台架构的二进制运行文件,大家可以根据自己的平台架构环境下载相应的文件  
https://developers.cloudflare.com/argo-tunnel/downloads   
  
下面是以ubuntu 18举例的配置流程,其他linux发行版可以依葫芦画瓢 
    
下载Cloudflare的二进制运行文件   
   
$su   
#cd /   
#wget https://bin.equinox.io/c/VdrWdbjqyF/cloudflared-stable-linux-amd64.tgz   
   
解压并拷贝到运行目录  
   
#tar xvf cloudflared-stable-linux-amd64.tgz  
  
#mv -f cloudflared /usr/bin/cloudflared  
  
检查运行权限,如果没有就置运行权限  
  
#ls -l /usr/bin/cloudflared  
  
运行程序验证可用  
  
#cloudflared --version  
  
建立配置目录并编辑配置文件   
  
#mkdir /etc/cloudflared/  
  
#nano /etc/cloudflared/config.yml  
  
写入下面内容,DoH服务器可以换成其他的,甚至是像IPv6的DoH服务器地址https://[2606:4700:4700::1001]/dns-query, 各个DoH服务商DoH的IP地址大家可以自己查,但是务必是查找服务商的IP已经直接签发IP证书的,就是可以直接https://ip 的这种,验证很简单,就是把这个地址按照https://ip/dns-query 格式放到浏览器浏览,如果正常出现小锁头而没有安全告警就是这个IP地址直接签发了证书,如果还用域名连接DoH查DNS的话,底层没有办法脱离原来DNS的限制,等于整台linux服务器还是受制于原来的DNS查询方式  
 　 
以下内容为了行首缩进,排版是使用了全角的空格,拷贝内容后务必替换成英文的两个空格　　

proxy-dns: true  
proxy-dns-port: 53  
proxy-dns-upstream:  
　- https://9.9.9.10/dns-query  
　- https://9.9.9.9/dns-query  
　- https://[2606:4700:4700::1001]/dns-query  
　- https://[2606:4700:4700::1111]/dns-query  
  
保存并退出  
  
安装服务,关键是带--legacy参数,否则出错,而且出错的提示和这个毫无关系,让你毫无头绪,而cloudflare网站是没有这个参数的介绍的  
   
#cloudflared service install --legacy  
  
设置开机启动服务  
systemctl enable cloudflared  
  
启动服务  
systemctl start cloudflared  
  
检查服务状态  
systemctl status cloudflared  
  
检查DNS监听服务器地址  
#nslookup www.163.com  
应该输出下面的DNS服务器地址信息,但这时系统里面的DNS服务器设置仍然是原来的  
  
Server:		127.0.0.53  
Address:	127.0.0.53#53  
  
然后修改netplan或network-manager的DNS服务器为127.0.0.1及"::1"如果有IPv6的DoH服务器设置的话,如果修改有疑问可以参考之前我另外一个repo的介绍    
  
修改完成后重启机器  
  
然后检查系统DNS服务器是否显示为127.0.0.1  
  
#systemd-resolve --status  
  

