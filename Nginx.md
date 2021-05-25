### Nginx

###### 为什么会使用到nginx

- 单个服务器访问量过大，需要使用多个服务器来缓解压力，而且访问的地址应该为同一个地址

准备工作：
	1.安装nginx
	①首先安装nginx前置软件
	yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
	②下载nginx软件安装包
	cd /data/tmp
	wget http://nginx.org/download/nginx-1.18.0.tar.gz
	tar -zxvf nginx-1.18.0.tar.gz
	cd nginx-1.18.0
	③设置安装目录为/usr/local/nginx
	./configure --prefix=/usr/local/nginx
	make&&make install

nginx常用命令:
	前提必须先进入nginx的安装目录的sbin中使用、
	1../nginx	启动nginx
	2../nginx -v	查看nginx版本信息
	3../nginx -s stop	关闭nginx
	4../nginx -s reload	重新加载配置文件

注意：使用新添加的nginx.conf启动一个新的nginx命令：/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx81.conf


反向代理：
	实例一:
	实现效果:使用nginx反向代理，访问www.123.com直接跳转到127.0.0.1:8080
	1.进入到nginx安装目录中的conf文件中找到nginx.conf文件	cd /usr/local/nginx/conf
	2.修改配置文件中的配置    将server_name修改为访问域名， 再在location中使用proxy_pass 代理地址;来进行反向代理
	3.重新加载nginx配置	./nginx -s reload
	

	实例二:
	实现效果:使用nginx反向代理，nginx监听端口9001，访问http://ip:9001/b/跳转到百度，访问http://ip:9001/bi/跳转到b站
	1.进入nginx安装目录中的conf中打开nginx.conf文件准备修改
	2.用已有的server或者新写一个server,在其中写入listen 端口（需要监听的端口），server_name localhost(访问的域名),编写location 匹配指令 {proxy_pass 代理地址} 来进行反向代理
	3.重新加载nginx配置文件	./nginx -s reload


负载均衡:
	实现效果:使用nginx负载均衡，同一个域名根据规则访问不同的地址
	1.进入nginx安装目录中的conf中打开nginx.conf文件准备修改
	2.在server前面添加 upstream upstream_name {} 来配置映射服务器，其中可以配置规则
	3.在server里面配置 proxy_pass http://upstream_name; 
	4.重新加载nginx配置	./nginx -s reload


动静分离:
	一种是在nginx.conf的location中配置静态资源的后缀[location ~.*\.(js|css)${root/opt/static}](了解)	另一种是在nginx.conf的location中配置静态资源所在目录[location ~.*/(css|js){root /opt/static}](常用)
	1.进入nginx安装目录中的conf中
	2.在nginx.conf中的server下新增一个location ~.*/(css|js|img){}
	两种方式:①直接在｛｝中写静态资源路径 ②使用负载均衡放在不同的nginx服务中（需要新增nginx.conf文件）
	3.使用①方式则跳过，使用②则复制nginx.conf并重新命名，在新的配置文件中配置第二步中的location
	4.使用 /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx81.conf -t 检查配置文件是否有问题
	5.使用 /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx81.conf 启动新的nginx
	

高可用性:
	使用keepalived实现
	1.安装keepalived软件	yum -y install keepalived	安装位置:/etc/keepalived
	2.修改keepalived配置文件
	注意：global_defs中需要配置router_id(访问服务器名称 /etc/hosts)	vrrp_script chk_http_port是检查脚本	vrrp_instance_VI_1是设置虚拟IP的信息，其中state是设置主|从，interface是设置网卡(ip addr)，virtual_router_id后面的值主从必须一致，priority是配置优先级，主大于从，virtual_ipaddress中是设置虚拟地址
	3.添加检测脚本
	4.启动nginx和keepalived		systemctl start keepalived.service 

虚拟主机:
	一种基于端口,域名一样但是端口不一样（了解）	另一种基于域名，端口一样，域名不一样，需要DNS绑定（掌握）
	1.进入nginx安装目录中的conf中打开nginx.conf文件准备修改
	2.新增一个server，将该server中的server_name修改为DNS绑定的域名
	3.为该server配置一个负载均衡，即upstream beijingweb {server ip:端口}
	4.在该server中的location里使用proxy_pass http://beijingweb;
	5.参照上面的步骤新增多个server和负载均衡
	6.重新加载nginx配置	./nginx -s reload
	7.实现本地进行DNS，在hosts中修改
	8.访问对应的域名实现进入不同的web