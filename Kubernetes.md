### K8S学习---kubernetes

#### 第一部分

1. k8s概述和特性

   **概述**

   - 集群：很多服务器集中起来一起进行同一种服务

   - k8s是个什么东西？能做什么事情？
   - k8s是谷歌在2014年开源的容器化集群管理系统
   - k8s可以用来进行容器化应用部署、使用k8s利于应用扩展
   - k8s的目的是让部署容器化应用更加简洁高效

   **特性**

   - 自动装箱：通过配置自动部署应用容器
   - 自我修复：容器失败会重启，部署节点有问题会重新部署和调度，只有通过检查才对外服务
   - 水平扩展：可以根据需求对容器规模进行调整
   - 服务发现：可以实现发配和负载均衡
   - 滚动更新：根据应用变化对容器中的应用进行更新
   - 版本回退：可以进行历史版本即时回退
   - 密钥和配置管理：类似热更新
   - 存储编排：自动实现存储系统的挂载和应用
   - 批处理：提供定时任务、一次性任务

2. k8s架构组件

   **Master(主控节点)**

   - API server：集群统一的入口，以restful方式，将结果交给etcd存储
   - scheduler：节点调度，选择node节点应用部署
   - controller-manager：处理集群中常规后台任务，一个资源操作对应一个控制器
   - etcd：存储系统，用于保存集群中的相关数据

   **Node(工作节点)**

   - kubelet：master派到node节点的代表，管理本节点容器
   - kube-proxy：提供网络代理，通过它可以实现负载均衡

3. k8s核心概念

   - pod：k8s中最小的单元，一组容器的集合，一个pod中的容器是共享网络的，生命周期是短暂的
   - controller：确保预期的pod副本数量、确保所有的node运行同一个pod、一次性任务和定时任务、通过controller创建pod
   - service：定义一组pod的访问规则

4. 整体流程：

   通过service统一入口进行访问，由controller创建Pod进行部署

#### 第二部分

1. 搭建K8S环境平台规划
   - 单Master集群：缺点是只要master挂掉了就无法使用了
   - 多Master集群：多master中需要在master和node之间多一个负载均衡的步骤（高可用）

2. 搭建K8S服务器硬件配置要求
   - 测试环境：Master至少2核4G20G存储，Node至少4核8G40G存储
   - 生产环境：比测试环境配置更高
3. 搭建K8S集群部署方式
   - kubeadm方式：使用Kubeadm这个K8S部署工具（kubeadm init|kubeadm join），特点就是方便快速[Kubeadm工具文档](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/)
   - 二进制包方式：手动下载包、手动部署，特点过程麻烦，部署慢，但是可以学习很多工作原理利于后期维护

#### 第三部分

**本地虚拟机使用kebeadm部署K8S**

**注意：**

1. 系统需要是centOS7.x
2. 需要满足配置要求
3. 集群中的所有机器之间网络互通
4. 集群中的所有机器可以访问外网，用于拉取镜像
5. 禁止swap分区

**步骤一：系统环境配置（所有机器都需要进行）**

1. 关闭防火墙：

   ```
   systemctl stop firewalld（临时）|systemctl disable firewalld（永久）
   ```

2. 关闭selinux：

   ```
   setenforce 0（临时）|sed -i 's/enforcing/disabled/' /etc/selinux/config（永久）
   ```

3. 关闭swap：

   ```
   swapoff -a（临时）|sed -ri 's/.*swap.*/#&/' /etc/fstab（永久）
   ```

4. 设置主机名（master和node）：

   ```
   hostnamectl set-hostname <hostname>
   eg:
   hostnamectl set-hostname k8smaster
   hostnamectl set-hostname k8snode1
   hostnamectl set-hostname k8snode2
   查看主机名
   hostname
   ```

5. 在Master主机中添加hosts

   ```
   在/ets/hosts文件中添加ip和主机名
   cat >> /etc/hosts << EOF
   192.168.44.141 k8smaster
   192.168.44.142 k8snode1
   192.168.44.143 k8snode2
   EOF
   ```

6. 将桥接的IPv4流量传递到iptables的链

   ```
   cat > /etc/sysctl.d/k8s.conf << EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   //添加完毕后使其生效
   sysctl --system
   ```

7. 时间同步

   ```
   //更新yum中的ntpdata包
   yum install ntpdate -y
   ntpdate time.windows.com
   ```

   

**步骤二：安装需要的软件（所有机器都需要进行）**

- 注意：kubernetes默认CRI（容器运行时）为Docker

1. 安装Docker

   ```
   //下载docker-ce.repo并放在/etc/yum.repos.d/docker-ce.repo
   wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -o /etc/yum.repos.d/docker-ce.repo
   //安装docker
   yum -y install docker-ce-18.06.1.ce-3.e17
   //将docker加入开启启动项并启动docker
   systemctl enable docker 
   systemctl start docker
   //查看docker是否成功安装
   docer --version
   //更改docker仓库
   cat > /etc/docker/daemon.json << EOF
   {
   	"registry-mirrors":["https://b9pmyelo.mirror.aliyumcs.com"]
   }
   EOF
   //改完后重启docker
   systemctl restart docker
   //查看是否更改成功
   docker info
   ```

2. 添加kubernetes的阿里云YUM软件源

   ```
   cat > /etc/yum.repos.d/kubernetes.repo << EOF
   [kubernetes]
   name=kubernetes
   baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-e17-x86_64
   enabled=1
   gpgcheck=0
   repo_gpgcheck=0
   gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
   https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
   EOF
   ```

3. 安装kubeadm、kubelet和kubectl

   ```
   yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
   //设置kubelet为开机自启项
   systemctl enable kubelet
   ```

**步骤三：Master节点设置**

1. 拉取镜像并做初始化

   ```
   kubeadm init \
   --apiserver-advertise-address=192.168.44.141 \ (此处地址为mater节点的IP)
   --image-repository registry.aliyuncs.com/google_containers \
   --kubernetes-version v1.18.0 \
   --service-cidr=10.96.0.0/12 \ （此处地址只要和节点IP不同即可）
   --pod-network-cidr=10.244.0.0/16 （此处地址只要和节点IP不同即可）
   //完整命令
   kubeadm init --apiserver-advertise-address=192.168.44.141 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12 --pod-network-cidr=10.244.0.0/16
   //上面命令执行完成后继续执行提示的命令
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   //查看node节点信息
   kubectl get nodes
   ```

**注意：**初始化时生成的token会过期（默认24小时），重新创建token操作如下

```
kubeadm token create --print-join-command
```

**步骤四：Node节点加入管理**

1. 将当前主机加入管理

   ```
   kubeadm join 192.168.1.11:6643 --token esce21.q6hetwm8si29qxwn \ (此处命令为初始化后提示)
   	--discovery-token-ca-cert-hash sha256:00603a05805807501d7181c3d60b478788408cfe6cedefedb1f97569708be9c5
   ```

**步骤五：配置网络插件（只需要在Master上安装）**

1. 下载插件

   ```
   wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   
   //使用kubectl下载
   kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   //查看
   kubectl get pods -n kube-system
   ```

**步骤六：创建一个pod做最基本的测试（在Master上进行）**

```
//下载一个nginx的镜像
kubectl create deployment nginx --image=nginx
//查看已有的pod
kubectl get pod
//使其对外暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
//查看节点对外暴露的端口
kubectl get pod,svc
```

#### 第四部分

**本地虚拟机部署K8S---二进制包**

**注意：**

1. 系统是centOS7.x
2. 系统配置需要满足要求
3. 各个服务器之间要网络连通
4. 各个服务器之间要能访问外网，需下载包
5. 禁止swap分区

**步骤一：系统初始化（Master和Node都需要做）**

1. 关闭防火墙

   ```
   systemctl stop firewalld(临时关闭)
   systemctl disable firewalld(永久关闭)
   ```

2. 关闭seLinux

   ```
   seteforce 0(临时关闭)
   sed -i 's/enforcing/disabled/' /etc/selinux/config(永久关闭)
   ```

3. 关闭swap分区

   ```
   swapoff -a(临时关闭)
   sed -ri 's/.*swap.*/#&/' /etc/fstab(永久关闭) 
   ```

4. 设置主机名

   ```
   hostnamectl set-hostname <hostname>
   ```

5. 在Master节点上修改hosts

   ```
   cat >> /etc/hosts << EOF
   192.168.44.147 m1
   192.168.44.148 n1
   EOF
   ```

6. 将桥接的IPv4流量传递到iptables的链（每个机器都要执行）

   ```
   cat > /etc/sysctl.d/k8s.conf << EOF
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   EOF
   //使其生效
   sysctl --system
   ```

7. 时间同步（每个机器都要执行）

   ```
   yum install ntpdate -y
   ntpdate time.woindows.com
   ```

   

**步骤二：为etcd和apiserver自签证书（随意一台机器）**

**注意：**master节点和node节点之间访问需要证书、外部访问集群也需要证书

1. 证书可以去kubernetes官网去签发
2.  证书也可以自己签发（学习使用）
3. 证书类型：cfssl（使用json文件生成证书）、openssl

**为etcd生成cfssl证书**

1. 下载证书

   ```
   wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
   wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
   wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
   ```

2. 给下载的包赋予权限

   ```
   chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
   ```

3. 移动包的位置

   ```
   mv cfssl_linux-adm64 /usr/local/bin/cfssl
   mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
   mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
   ```

4. 创建etcd、k8s文件夹

   ```
   mkdir -p TLS/{etcd,k8s}
   ```

5. 进入etcd并自签CA

   ```
   //创建ca-config.json文件
   cat > ca-config.json << EOF
   {
   	"signing":{
   		"default":{
   		"expiry":"97600h"
   		},
   		"profiles":{
   			"www":{
   				"expiry":"87600h",
   				"usages":[
   					"signing"
   					"key encipherment",
   					"server auth",
   					"client auth"
   				]
   			}
   		}
   	}
   }
   EOF
   
   //创建ca-csr.json文件
   cat > ca-csr.json << EOF
   {
   	"CN":"etcd CA",
   	"key":{
   		"algo":"rsa",
   		"size":2048
   	},
   	"names":[
   		{
   			"C":"CN",
   			"L":"SiChuang",
   			"ST":"SiChuang"
   		}
   	]
   }
   EOF
   
   //使用下载的命令cfssl和cfssljson根据上面两个文件生成证书
   cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
   //查看生成的证书(ca-key.pem、ca.pem)
   ls *pem
   ```

6. 使用自签CA签发etcd的HTTPS证书

   ```
   //创建证书申请文件
   cat > server-csr.json << EOF
   {
   "CN":"etcd",
   "hosts":[
   "192.168.44.147",//改成自己的master节点地址
   "192.168.44.148"//改成自己的node节点地址
   ],
   "key":{
   "algo":"rsa",
   "size":2048
   },
   "names":[
   {
   "C":"CN",
   "L":"SiChuang",
   "ST":"SiChuang"
   }
   ]
   }
   EOF
   
   //使用命令生成证书
   cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson
   //查看生成的证书(server-key.pem、server.pem)
   ls server*pem
   ```

   

**步骤三：部署etcd集群**

```
//下载etcd包
wget https://github.com/etcd-io/etcd/releases/download/v3.4.9/etcd-v3.4.9-
linux-amd64.tar.gz

//创建文件夹
mkdir /etcd/{bin,cfg,ssl} –p

//解压etcd包
tar zxvf etcd-v3.4.9-linux-amd64.tar.gz

//将解压文件移动到bin目录中
mv etcd-v3.4.9-linux-amd64/{etcd,etcdctl} /etcd/bin/

//创建etcd.service文件
cat > etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
EnvironmentFile=/opt/etcd/cfg/etcd.conf
ExecStart=/opt/etcd/bin/etcd \
		--name=${ETCD_NAME} \
		--data-dir=${ETCD_DATA_DIR} \
		--listen-peer-urls=${ETCD_LISTEN_PEER_URLS} \
		--listen-client-urls=${ETCD_LISTEN_CLIENT_URLS},http://127.0.0.1:2379 \
		--advertist-client-urls=${ETCD_ADVERTISE_CLIENT_URLS} \
		--initial-advertise-peer-urls=${ETCD_INITIAL_ADVERTISE_PEER_URLS} \
		--initial-cluster=${ETCD_INITIAL_CLUSTER} \
		--initial-cluster-token=${ETCD_INITIAL_CLUSTER_TOKEN} \
		--initial-cluster-state=new \
		--cert-file=/opt/etcd/ssl/server.pem \
		--key-file=/opt/etcd/ssl/server-key.pem \
		--peer-cert-file=/opt/etcd/ssl/server.pem \
		--peer-key-file=/opt/etcd/ssl/server-key.pem \
		--trusted-ca-file=/opt/etcd/ssl/ca.pem \
		--peer-trusted-ca-file=/opt/etcd/ssl/ca.pem 
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF

//将etcd.service文件放到对应的目录中
mv etcd.service /usr/lib/systemd/system/

//bin目录里面为可执行文件、ssl中需要放置证书文件、cfg中放置etcd.conf文件
//进入cfg目录中创建etcd.conf文件
cat > etcd.conf << EoF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.44.147:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.44.147:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.44.147:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.44.147:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.44.147:2380,etcd-2=https://192.168.44.148:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF

//将ca.pem、server-key.pem、server.pem放置到ssl中
cp 用于生成证书的etcd文件夹路径/{ca,server,server-key} /用于部署etcd集群创建的etcd路径/ssl/
//将生成的证书文件放到上面etcd.service中指定的位置
mv etcd/ /opt
//将opt中的etcd文件发送到其他所有的节点中
scp -r /opt/etcd/ root@192.168.44.148:/opt/

//将etcd.service文件发送到其他所有节点机器中
scp /usr/lib/systemd/system/etcd.service root@192.168.44.148:/usr/lib/systemd/system/

//修改其他节点上cfg中etcd.conf文件
①ETCD_NAME  ②ETCD_LISTEN_PEER_URLS  ③ETCD_LISTEM_CLIENT_URLS ④ETCD_INITIAL_ADVERTISE_PEER_URLS ⑤ETCD_ADVERTISE_CLIENT_URLS

//先在Master节点上使用
systemctl daemon-reload
systemctl start etcd

//再在node节点上使用
systemctl daemon-reload
systemctl start etcd

//查看状态
systemctl status etcd.servicd
```

**步骤四：为ApiServer自签证书**(Master节点)

1. APIserver是通过https方式进行访问的，需要有证书才能正常访问
2. 方式：①添加可信任的IP列表    ②发送请求时都带ca证书发送

```
//进入到生成ectd集群证书时创建的K8S目录并生成server-csr.json、kube-proxy-csr.json、ca-config.json、ca-csr.json文件

cat > kube-proxy-csr.json << EOF
{
	"CN":"system:kube-proxy",
	"hosts":[],
	"key":{
		"algo":"rsa",
		"size":2048
	}
	"names":[
		{
			"C":"CN",
			"L":"SiChuang",
			"ST":"SiChuang",
			"O":"k8s",
			"OU":"System"
		}	
	]
}
EOF
//添加可信任列表
cat > server-csr.json << EOF
{
	"CN":"kubernetes",
	"hosts":[
		"10.0.0.1",
		"127.0.0.1",
		"kubernetes",
		"kubernetes.default",
		"kubernetes.default.svc",
		"kubernetes.default.svc.cluster",
		"kunernetes.default.svc.cluster.local",
		"192.168.44.147",
		"192.168.44.148"
	],
	"key":{
		"algo":"res",
		"size":2048
	},
	"names":[
		{
		"C":"CN",
		"L":"SiChuang",
		"ST":"SiChuang",
		"O":"k8s",
		"OU":"System"
		}
	]
}
EOF
//下面这两个文件是为了生成ca-key.pem和ca.pem
cat > ca-config.json<< EOF
{
"signing": {
"default": {
"expiry": "87600h"
},
"profiles": {
"kubernetes": {
"expiry": "87600h",
"usages": [
"signing",
"key encipherment",
"server auth",
"client auth"
]
}
}
}
}
EOF

cat > ca-csr.json<< EOF
{
"CN": "kubernetes",
"key": {
"algo": "rsa",
"size": 2048
},
"names": [
{
"C": "CN",
"L": "Beijing",
"ST": "Beijing",
"O": "k8s",
"OU": "System"
}
]
}
EOF

//执行命令生成自签证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -
profile=kubernetes server-csr.json | cfssljson -bare server
```

