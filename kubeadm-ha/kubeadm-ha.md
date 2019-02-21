

[TOC]



# Ŀ¼

## ����Kubeadm����Kubernetes1.13.3 HA �߿��ü�Ⱥ 

> K8S 1.13�汾��Ĭ�ϣ�
>
> ?	DNS��CoreDNS
>
> ?        Kube-proxy�� IPVSģ��    #��iptables
>
> ?        Network�� Calico
>
> ע�⣺
>
> Ĭ��kubeadmһ������`etcd`�Ǽ�Ⱥ�߿��ã�
>
> ��ʾ��ʹ��`���etcd`ʵ�ָ߿��ü�Ⱥ��Master APIserverʹ��Keeplived��

### 01. ����Ŀ��

#### 1.1 Kubernetes������

- �ֲ�ʽ����
- ������
- ����ʵ������
- ��������
- �ڵ��޺�ά��
- ����ƽ��
- ��̬������

�Ӷ��ܹ�����δ��΢������ά��������

#### 1.2 ��΢���񣬿����������ٲ���

> 01ͨ��docker���񣬿��Կ��ٲ���һ�µ�Ϊÿ��������Ա�ṩ��ͬ��linux����������ʡȥ��ÿ��Ա�����в��𿪷��������������ʺ����࣬�ÿ�����Ա�ܹ���רע��code������ʡ����ʱ��



### 02. ����˵��

- ��̨�������в���K8S v1.13+��Ⱥ������һ̨����Harbor������`etcd`Ϊ���нڵ㲿��
- Kubernetes���������ݶ��Ǵ洢��etcd�еģ�etcd����߿��ü�Ⱥ
- Masterʹ��keepalived�߿��ã�Master��Ҫ�Ƿַ��û�����ָ��Ȳ�����
- Master�ٷ���������keepalived���м�Ⱥ������Ҳ����ʹ���Խ�LB/��ҵAWS�ģ�ALB ELB ��

|            System             |   Roles   |  IP Address  |
| :---------------------------: | :-------: | :----------: |
| CentOS Linux release 7.4.1708 | Master01  | 172.16.1.50  |
| CentOS Linux release 7.4.1708 | Master02  | 172.16.1.51  |
| CentOS Linux release 7.4.1708 |  Node01   | 172.16.1.52  |
| CentOS Linux release 7.4.1708 |  Node02   | 172.16.1.53  |
| CentOS Linux release 7.4.1708 |  Node01   | 172.16.1.54  |
| CentOS Linux release 7.4.1708 | VM Harbor | 172.16.0.181 |

#### 2.1 ��Ⱥ˵��

|      Software       | Version |
| :-----------------: | :-----: |
|     Kubernetes      | 1.13.3  |
|      Docker-CE      | 18.06.1 |
|        Etcd         | 3.3.11  |
|       Calico        |  3.1.4  |
|      Dashboard      | v1.10.0 |
|   Traefik Ingress   |  1.7.9  |
| prometheus-operator | 0.29.0  |



### 03. K8S��Ⱥ����˵��

#### 3.1 Kubernetes

>  Kubernetes �� Google �Ŷӷ���ά���Ļ���Docker�Ŀ�Դ������Ⱥ����ϵͳ��������֧�ֳ�������ƽ̨������֧���ڲ��������ġ����� Docker ֮�ϵ� Kubernetes ���Թ���һ�������ĵ��ȷ�����Ŀ�������û�͸��Kubernetes��Ⱥ�������ƶ�������Ⱥ�Ĺ����������û����и��ӵ����ù�����ϵͳ���Զ�ѡȡ���ʵĹ����ڵ���ִ�о����������Ⱥ���ȴ�����������ĸ�����Container Pod�������֣���һ��Pod����һ�鹤����ͬһ�������ڵ���������ɵġ���Щ������ӵ����ͬ�����������ռ�/IP�Լ��洢�����Ը���ʵ�������ÿһ��Pod���ж˿�ӳ�䡣���⣬Kubernetes�����ڵ������ϵͳ���й����ڵ�������ܹ�����Docker�������õ��ķ���

####  3.2 Docker

>  Docker��һ����Դ�����棬�������ɵ�Ϊ�κ�Ӧ�ô���һ���������ġ�����ֲ�ġ��Ը�������������������ڱʼǱ��ϱ������ͨ�����������������������������в��𣬰���VMs�����������bare metal��OpenStack ��Ⱥ�������Ļ���Ӧ��ƽ̨��

####  3.3 Etcd

> ETCD�����ڹ������úͷ����ֵķֲ�ʽ��һ���Ե�KV�洢ϵͳ��

#### 3.4 Calico

> ͬFlannel,���ڽ�� docker ����ֱ�ӿ�������ͨ�����⣬Calico��ʵ���Ƿ�����ʽ�ģ�����������ֱ��ͨ��iptablesת��������û�����ģ�flannel��Ҫ����������cpu���ģ�Ч�ʲ���calico��calico������ԭ������� 



### 04. ��ʼ����Kubernetes��Ⱥ

#### 4.1 ��װǰ׼��

����2019��2�£�KubernetesĿǰ�ĵ��汾��v1.13+  �ٷ��汾�����ܿ죬����ѡ��Ŀǰ�ĵ��汾�

**K8S���нڵ�����������**

```shell
# ����������
hostnamectl set-hostname K8S01-Master01
hostnamectl set-hostname K8S01-Master02
hostnamectl set-hostname K8S01-Node01
hostnamectl set-hostname K8S01-Node02
hostnamectl set-hostname K8S01-Node03

# ����hosts
cat <<EOF > /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
172.16.1.50 master01 K8S01-Master01
172.16.1.51 master02 K8S01-Master02
172.16.1.52 node01 K8S01-Node01
172.16.1.53 node02 K8S01-Node02
172.16.1.54 node03 K8S01-Node03
EOF

#��������Կ��½
ssh-keygen   #һֱ�س�
ssh-copy-id   master01
ssh-copy-id   master02
ssh-copy-id   node01
ssh-copy-id   node02
```



#### 4.2 �Ż�ϵͳ�ͼ�Ⱥ׼��

```shell
#�رշ���ǽ
systemctl stop firewalld
systemctl disable firewalld

###�ر�Swap
swapoff -a 
sed -i 's/.*swap.*/#&/' /etc/fstab

###����Selinux
setenforce  0 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/sysconfig/selinux 
sed -i "s/^SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config 
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/sysconfig/selinux 
sed -i "s/^SELINUX=permissive/SELINUX=disabled/g" /etc/selinux/config  

###������ο����汨����
modprobe br_netfilter   
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
EOF
sysctl -p /etc/sysctl.d/k8s.conf
ls /proc/sys/net/bridge

###K8SԴ
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

###�ں��Ż�
echo "* soft nofile 204800" >> /etc/security/limits.conf
echo "* hard nofile 204800" >> /etc/security/limits.conf
echo "* soft nproc 204800"  >> /etc/security/limits.conf
echo "* hard nproc 204800"  >> /etc/security/limits.conf
echo "* soft  memlock  unlimited"  >> /etc/security/limits.conf
echo "* hard memlock  unlimited"  >> /etc/security/limits.conf

###kube-proxy����ipvs��ǰ������
# ԭ�ģ�https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/ipvs/README.md
# �ο���https://www.qikqiak.com/post/how-to-use-ipvs-in-kubernetes/

# ����ģ�� <module_name>
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4

# �����ص�ģ��
lsmod | grep -e ipvs -e nf_conntrack_ipv4
# ����
cut -f1 -d " "  /proc/modules | grep -e ip_vs -e nf_conntrack_ipv4
```



#### 4.3 ��װDocker-CE

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

yum makecache fast
yum install -y --setopt=obsoletes=0 \
  docker-ce-18.06.1.ce-3.el7

systemctl start docker
systemctl enable docker
```



#### 4.4 ���нڵ�����Docker������� 

����������������������õ�ַ<https://dev.aliyun.com/search.html>  ��¼�������Ļ�ȡ����ר����������ַ

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://3csy84rx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```





### 05. ����TLS֤�����Կ

#### 5.1 Kubernetes ��Ⱥ����֤��

> `ca`֤��Ϊ��Ⱥadmin֤�顣
>
> `etcd`֤��Ϊetcd��Ⱥʹ�á�
>
> `shinezone`֤��ΪHarborʹ�á�

|      CA&Key       | etcd | api-server | proxy | kebectl | Calico | harbor |
| :---------------: | :--: | :--------: | :---: | :-----: | :----: | :----: |
|      ca.csr       |  ��   |     ��      |   ��   |    ��    |   ��    |        |
|      ca.pem       |  ��   |     ��      |   ��   |    ��    |   ��    |        |
|    ca-key.pem     |  ��   |     ��      |   ��   |    ��    |   ��    |        |
|      ca.pem       |  ��   |            |       |         |        |        |
|     etcd.csr      |  ��   |            |       |         |        |        |
|   etcd-key.pem    |  ��   |            |       |         |        |        |
| shinezone.com.crt |      |            |       |         |        |   ��    |
| shinezone.com.key |      |            |       |         |        |   ��    |



#### 5.2 ��װCFSSL

- K8S01ִ��

```
yum install wget -y
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
chmod +x cfssl_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
chmod +x cfssljson_linux-amd64
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
chmod +x cfssl-certinfo_linux-amd64
mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
export PATH=/usr/local/bin:$PATH
```

#### 5.3 ����CA�ļ�,����etcd֤��

```
mkdir /root/ssl
cd /root/ssl
cat >  ca-config.json <<EOF
{
"signing": {
"default": {
  "expiry": "8760h"
},
"profiles": {
  "kubernetes-Soulmate": {
    "usages": [
        "signing",
        "key encipherment",
        "server auth",
        "client auth"
    ],
    "expiry": "8760h"
  }
}
}
}
EOF

cat >  ca-csr.json <<EOF
{
"CN": "kubernetes-Soulmate",
"key": {
"algo": "rsa",
"size": 2048
},
"names": [
{
  "C": "CN",
  "ST": "shanghai",
  "L": "shanghai",
  "O": "k8s",
  "OU": "System"
}
]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

#hosts����Ҫ��������etcd��Ⱥ�ڵ㣬���齫����nodeҲ���룬��������etcd��Ⱥ��
cat > etcd-csr.json <<EOF
{
  "CN": "etcd",
  "hosts": [
    "127.0.0.1",
    "172.16.1.50",
    "172.16.1.51",
    "172.16.1.52",
    "172.16.1.53",
    "172.16.1.54"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "shanghai",
      "L": "shanghai",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF

cfssl gencert -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes-Soulmate etcd-csr.json | cfssljson -bare etcd
```

�ֶ�˵��

- ��� hosts �ֶβ�Ϊ������Ҫָ����Ȩʹ�ø�֤���?**IP �������б�**

- `ca-config.json`�����Զ����� profiles���ֱ�ָ����ͬ�Ĺ���ʱ�䡢ʹ�ó����Ȳ�����������ǩ��֤��ʱʹ��ĳ�� profile��
- `signing`����ʾ��֤�������ǩ������֤�飻���ɵ� ca.pem ֤����?`CA=TRUE`��
- `server auth`����ʾclient�����ø� CA ��server�ṩ��֤�������֤��
- `client auth`����ʾserver�����ø�CA��client�ṩ��֤�������֤��
- "CN"��`Common Name`��kube-apiserver ��֤������ȡ���ֶ���Ϊ������û��� (User Name)�������ʹ�ø��ֶ���֤��վ�Ƿ�Ϸ���
- "O"��`Organization`��kube-apiserver ��֤������ȡ���ֶ���Ϊ�����û��������� (Group)��



#### 5.4 �ַ�֤�鵽���нڵ�

- Master01ִ��

> ����Ⱥ�������нڵ㰲װetcd�������Ҫ֤��ַ����нڵ㡣

```shell
mkdir -p /etc/etcd/ssl
cp etcd.pem etcd-key.pem ca.pem /etc/etcd/ssl/
scp -r /etc/etcd/ master02:/etc/
scp -r /etc/etcd/ node01:/etc/
scp -r /etc/etcd/ node02:/etc/
scp -r /etc/etcd/ node03:/etc/
```

### 06. ��װ����etcd

#### 6.1 ��װetcd

- ���нڵ�ִ��

```shell
yum install etcd -y   
mkdir -p /var/lib/etcd
```

#### 6.2 ����etcd

master01��`etcd.service`

```shell
cat <<EOF >/usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/bin/etcd \
  --name k8s01 \
  --cert-file=/etc/etcd/ssl/etcd.pem \
  --key-file=/etc/etcd/ssl/etcd-key.pem \
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \
  --trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --initial-advertise-peer-urls https://172.16.1.50:2380 \
  --listen-peer-urls https://172.16.1.50:2380 \
  --listen-client-urls https://172.16.1.50:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://172.16.1.50:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster k8s01=https://172.16.1.50:2380,k8s02=https://172.16.1.51:2380,k8s03=https://172.16.1.52:2380,k8s04=https://172.16.1.53:2380,k8s05=https://172.16.1.54:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

master02��`etcd.service`

```shell
cat <<EOF >/usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/bin/etcd \
  --name k8s02 \
  --cert-file=/etc/etcd/ssl/etcd.pem \
  --key-file=/etc/etcd/ssl/etcd-key.pem \
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \
  --trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --initial-advertise-peer-urls https://172.16.1.51:2380 \
  --listen-peer-urls https://172.16.1.51:2380 \
  --listen-client-urls https://172.16.1.51:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://172.16.1.51:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster k8s01=https://172.16.1.50:2380,k8s02=https://172.16.1.51:2380,k8s03=https://172.16.1.52:2380,k8s04=https://172.16.1.53:2380,k8s05=https://172.16.1.54:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

node01��`etcd.service`

```shell
cat <<EOF >/usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/bin/etcd \
  --name k8s03 \
  --cert-file=/etc/etcd/ssl/etcd.pem \
  --key-file=/etc/etcd/ssl/etcd-key.pem \
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \
  --trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --initial-advertise-peer-urls https://172.16.1.52:2380 \
  --listen-peer-urls https://172.16.1.52:2380 \
  --listen-client-urls https://172.16.1.52:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://172.16.1.52:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster k8s01=https://172.16.1.50:2380,k8s02=https://172.16.1.51:2380,k8s03=https://172.16.1.52:2380,k8s04=https://172.16.1.53:2380,k8s05=https://172.16.1.54:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

node02��`etcd.service`

```shell
cat <<EOF >/usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/bin/etcd \
  --name k8s04 \
  --cert-file=/etc/etcd/ssl/etcd.pem \
  --key-file=/etc/etcd/ssl/etcd-key.pem \
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \
  --trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --initial-advertise-peer-urls https://172.16.1.53:2380 \
  --listen-peer-urls https://172.16.1.53:2380 \
  --listen-client-urls https://172.16.1.53:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://172.16.1.53:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster k8s01=https://172.16.1.50:2380,k8s02=https://172.16.1.51:2380,k8s03=https://172.16.1.52:2380,k8s04=https://172.16.1.53:2380,k8s05=https://172.16.1.54:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

node03��`etcd.service`

```shell
cat <<EOF >/usr/lib/systemd/system/etcd.service
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/bin/etcd \
  --name k8s05 \
  --cert-file=/etc/etcd/ssl/etcd.pem \
  --key-file=/etc/etcd/ssl/etcd-key.pem \
  --peer-cert-file=/etc/etcd/ssl/etcd.pem \
  --peer-key-file=/etc/etcd/ssl/etcd-key.pem \
  --trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ssl/ca.pem \
  --initial-advertise-peer-urls https://172.16.1.54:2380 \
  --listen-peer-urls https://172.16.1.54:2380 \
  --listen-client-urls https://172.16.1.54:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://172.16.1.54:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster k8s01=https://172.16.1.50:2380,k8s02=https://172.16.1.51:2380,k8s03=https://172.16.1.52:2380,k8s04=https://172.16.1.53:2380,k8s05=https://172.16.1.54:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

#### 6.3  ����etcd

```shell
 systemctl daemon-reload
 systemctl enable etcd
 systemctl start etcd
 systemctl status etcd
```

#### 6.4 ��Ⱥ״̬���(ά��)

> ʹ��v3�汾API

```shell
$ echo "export ETCDCTL_API=3" >>/etc/profile  && source /etc/profile
$ etcdctl version
etcdctl version: 3.2.18
API version: 3.2
```

> �鿴��Ⱥ����״̬

```shell
$ etcdctl --endpoints=https://172.16.1.50:2379,https://172.16.1.51:2379,https://172.16.1.52:2379,https://172.16.1.53:2379,https://172.16.1.54:2379 --cacert=/etc/etcd/ssl/ca.pem   --cert=/etc/etcd/ssl/etcd.pem   --key=/etc/etcd/ssl/etcd-key.pem   endpoint health
//�����Ϣ���£�
https://172.16.1.54:2379 is healthy: successfully committed proposal: took = 1.911784ms
https://172.16.1.50:2379 is healthy: successfully committed proposal: took = 2.648385ms
https://172.16.1.52:2379 is healthy: successfully committed proposal: took = 3.472479ms
https://172.16.1.51:2379 is healthy: successfully committed proposal: took = 2.850887ms
https://172.16.1.53:2379 is healthy: successfully committed proposal: took = 3.711259ms
```

> ��ѯ����key

```shell
$ etcdctl --endpoints=https://172.16.1.50:2379,https://172.16.1.51:2379,https://172.16.1.52:2379,https://172.16.1.53:2379,https://172.16.1.54:2379 --cacert=/etc/etcd/ssl/ca.pem   --cert=/etc/etcd/ssl/etcd.pem   --key=/etc/etcd/ssl/etcd-key.pem    get / --prefix --keys-only

// kubeadm��ʼ��֮ǰ��û���κ���Ϣ�ģ���ʼ����ɺ��ѯ�õ�����Ϣ�磺
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.autoscaling
/registry/apiregistration.k8s.io/apiservices/v1.batch
........................
```

> ���`����/ָ��`key(**���ɻ�������**)
>
> ���ϻ�������k8s�����������,��Ҫ����ض�����key�������������

```shell
 etcdctl --endpoints=https://172.16.1.50:2379,https://172.16.1.51:2379,https://172.16.1.52:2379,https://172.16.1.53:2379,https://172.16.1.54:2379 --cacert=/etc/etcd/ssl/ca.pem   --cert=/etc/etcd/ssl/etcd.pem   --key=/etc/etcd/ssl/etcd-key.pem    del / --prefix 
```





### 07 ��װ����Kubeadm

> ��Ⱥ���нڵ㰲װ`kebelet` `kubeadm` `kebectl`

```shell
yum install -y kubelet kubeadm kubectl
###�ݲ�������δ��ʼ��ǰ����Ҳ�ᱨ��
systemctl enable kubelet   
```

#### 7.1 ����kubelet

> ���нڵ��޸ģ�kubelet����Agent��ÿ̨Node�ϱ���Ҫ��װ�����

```shell
// ���л���ִ��
sed -i s#systemd#cgroupfs#g /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
echo 'Environment="KUBELET_EXTRA_ARGS=--v=2 --fail-swap-on=false --pod-infra-container-image=harbor.domain.com/shinezonetest/pause-amd64:3.1"' >>  /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
```

#### 7.2 ���������ļ�

```shell
systemctl daemon-reload
systemctl enable kubelet
```



### 08 Master�ڵ�߿���

> Master�ڵ�߿���ʹ��keepalived��Ҳ����ʹ����ҵELB ALB SLB�����Խ�N'gin'x���ؾ��⡣

#### 8.1 ��װkeepalived

- master�ڵ�ִ��

```shell
yum install -y keepalived
systemctl enable keepalived
```

#### 8.2 ����Keepalived

> ע���޸�`interface`��������`priority`Ȩ��ֵ��`unicast_peer`

 Master01 �����ļ�

```shell
cat <<EOF >/etc/keepalived/keepalived.conf
global_defs {
   router_id LVS_k8s
}

vrrp_script CheckK8sMaster {
    script "curl -k https://172.16.1.49:6443"    #VIP Address
    interval 3
    timeout 9
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens32       #Your Network Interface Name
    virtual_router_id 61
    priority 120          #Ȩ�أ����ִ��Ϊ��������һ����ѡ���һ̨ΪMaster
    advert_int 1
    mcast_src_ip 172.16.1.50  #local IP
    nopreempt
    authentication {
        auth_type PASS
        auth_pass sqP05dQgMSlzrxHj
    }
    unicast_peer {
        #172.16.1.50
        172.16.1.51    #����һ̨masterIP
    }
    virtual_ipaddress {
        172.16.1.49/24    # VIP
    }
    track_script {
        CheckK8sMaster
    }

}
EOF
```

Master02�����ļ�

```shell
cat <<EOF >/etc/keepalived/keepalived.conf
global_defs {
   router_id LVS_k8s
}

vrrp_script CheckK8sMaster {
    script "curl -k https://172.16.1.49:6443"
    interval 3
    timeout 9
    fall 2
    rise 2
}

vrrp_instance VI_1 {
    state MASTER
    interface ens32
    virtual_router_id 61
    priority 110
    advert_int 1
    mcast_src_ip 172.16.1.51
    nopreempt
    authentication {
        auth_type PASS
        auth_pass sqP05dQgMSlzrxHj
    }
    unicast_peer {
        172.16.1.50
        #172.16.1.51
    }
    virtual_ipaddress {
        172.16.1.49/24
    }
    track_script {
        CheckK8sMaster
    }

}
EOF
```

#### 8.3 ����Keepalived

```shell
sed s#'KEEPALIVED_OPTIONS="-D"'#'KEEPALIVED_OPTIONS="-D -d -S 0"'#g /etc/sysconfig/keepalived -i   //������־�ļ�
echo "local0.*    /var/log/keepalived.log" >> /etc/rsyslog.conf
service rsyslog restart
systemctl start keepalived
systemctl status keepalived
```

#### 8.4 ����Keepalived������

> ���ԣ��ر�һ̨Master��������IP�Ƿ�Ư�ƣ�API�Ƿ���á�

```shell
//ȷ��VIP��Master01��
$ ip a | grep inet |grep "172.16"
    inet 172.16.1.50/21 brd 172.16.7.255 scope global ens32
    inet 172.16.1.49/24 scope global ens32   //VIP
    
// �ر�Master01������ȷ��VIP�Ƿ�Ʈ��
$ ip a |grep inet |grep "172.16"
    inet 172.16.1.51/21 brd 172.16.7.255 scope global ens32
    inet 172.16.1.49/24 scope global ens32  //���Կ���˲���ƫ�Ƶ���Master02������
    
// ȷ��APi���������,Ҳ�����²���ʼ����Ⱥ����ԣ�ֱ�ӷ���dashboard��Ч����
$ curl https://your_dashboard_address/ -I
HTTP/1.1 200 OK
Server: nginx/1.10.0
Date: Wed, 06 Jun 2018 05:58:22 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 990
Connection: keep-alive
Accept-Ranges: bytes
Cache-Control: no-store
Last-Modified: Tue, 13 Feb 2018 11:17:03 GMT
```

### 09 ��ʼ����Ⱥ

> Master������ӳ�ʼ�������ļ�, V1.13�汾��version�汾ʹ��v1beta1 �﷨�仯�ܴ�

```shell
#cat EOF��ʽ��ʽ���ҵ���ֱ��vim ����ճ����ȥ�����ָ�ʽ����
#vim config.yaml   :set paste  

apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: "ipvs"  #ʹ��IPVSģʽ����iptables
---
apiVersion: kubeadm.k8s.io/v1beta1  #v1beta1�汾,��v1alpha�汾���﷨���б仯
certificatesDir: /etc/kubernetes/pki   
clusterName: kubernetes
controlPlaneEndpoint: 172.16.1.49:6443  #Keeplived ����IP��ַ
controllerManager: {}
dns:
  type: CoreDNS  #Ĭ��DNS��CoreDNS
imageRepository: k8s.gcr.io   #�ٷ�����
#imageRepository: harbor.domain.com/k8s   #���޸�Ϊ�Լ���Har�����
kind: ClusterConfiguration
kubernetesVersion: v1.13.3   #K8S�汾
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12  #SVC����� 
  podSubnet: 100.64.0.0/10     #POD�����
apiServer:
        certSANs:  
        - 172.16.1.50
        - 172.16.1.51
        - 172.16.1.52
        - 172.16.1.53
        - 172.16.1.54
        extraArgs: 
           etcd-cafile: /etc/etcd/ssl/ca.pem
           etcd-certfile: /etc/etcd/ssl/etcd.pem
           etcd-keyfile: /etc/etcd/ssl/etcd-key.pem
etcd:  #ʹ�����etcd�߿���
    external:
        caFile: /etc/etcd/ssl/ca.pem
        certFile: /etc/etcd/ssl/etcd.pem
        keyFile: /etc/etcd/ssl/etcd-key.pem
        endpoints:
        - https://172.16.1.50:2379
        - https://172.16.1.51:2379
        - https://172.16.1.52:2379
        - https://172.16.1.53:2379
        - https://172.16.1.54:2379
```



#### 9.1 ��ʼ����Ⱥ

- Master01����

```shell
kubeadm init --config config.yaml  

# ��ʼ�������п���journalctl -u kubelet -f�鿴log�����ܻᱨ��cni�������⣬��Ϊ����ָ�������磬calico��û��װ�����Ա�����calico��װ��ɾͺ���
```

��ʼ���ɹ��������Ϣ��

```shell
Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 172.16.1.49:6443 --token b99a00.a144ef80536d4344 --discovery-token-ca-cert-hash sha256:8c
```

��ʼ����ɺ�ִ���������

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

�鿴״̬

```shell
$ kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
etcd-2               Healthy   {"health":"true"}   
etcd-3               Healthy   {"health":"true"}   
etcd-4               Healthy   {"health":"true"}   
etcd-1               Healthy   {"health":"true"}
```

> ע�⣬��̨Master��������Ҫ���г�ʼ�����ڶ�̨master��ʼ��Ҳ��Ҫconfig.yaml�ļ���ǰ����Ҫ��

- Master02����

```shell
#�ַ�kebeadm���ɵ�֤���ļ��������ļ�
#ÿ̨Master������֤��������ļ�������ͬ�ģ����µ�Master���룬ֱ�ӷַ���ʼ�����ɡ�

$ scp -r /etc/kubernetes/pki  master02:/etc/kubernetes/
//Ȼ���ʼ����ִ������
$ kubeadm init --config config.yaml 
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 9.2 ��ʼ��ʧ�ܽ���취

```shell
$ kubeadm reset
// ����ɾ������ļ���images
$ rm -rf /etc/kubernetes/*.conf
$ rm -rf /etc/kubernetes/manifests/*.yaml
$ docker ps -a |awk '{print $1}' |xargs docker rm -f
$ systemctl  stop kubelet
```

�ٴγ�ʼ��ǰ��Ҫִ�����etcd�������ݵĲ�����

```shell
 $ etcdctl --endpoints=https://172.16.1.50:2379,https://172.16.1.51:2379,https://172.16.1.52:2379,https://172.16.1.53:2379,https://172.16.1.54:2379 --cacert=/etc/etcd/ssl/ca.pem   --cert=/etc/etcd/ssl/etcd.pem   --key=/etc/etcd/ssl/etcd-key.pem    del / --prefix
```

####

### 10. �����������

> Flanneld��Calico���ǽ������ͨ���������ѡһ�����ɣ�����ʹ��DaemonSet����ֻ��Master01ִ��

#### 10.1 Calico������𣨶�ѡһ��

```shell
wget https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/rbac-kdd.yaml
wget  https://docs.projectcalico.org/v3.1/getting-started/kubernetes/installation/hosted/kubernetes-datastore/calico-networking/1.7/calico.yaml
#�޸�calico.yaml��image����·������,��������harbor,�����ӵĿ���Ĭ��ֱ��apply yaml�ļ�
$ grep image calico.yaml 

        - image: harbor.domain.com/shinezonetest/calico-typha:1.0
          image: harbor.domain.com/shinezonetest/calico-node:1.0
          image: harbor.domain.com/shinezonetest/calico-cni:1.0
          
$ kubectl apply -f rbac-kdd.yaml
$ kubectl apply -f calico.yaml
```

#### 10.2 Flannel�������(��ѡһ)

- Ŀǰ����ʹ��Calico���磬��ѡһ���ɣ�Flannel����Ҫ����

```shell
$ mkdir -p /run/flannel/
$ cat >/run/flannel/subnet.env <<EOF
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
EOF
$ wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  #�汾��Ϣ��quay.io/coreos/flannel:v0.10.0-amd64
$ kubectl create -f  kube-flannel.yml
```

#### 10.3 �鿴��Ⱥ״̬

> ��Ⱥ�����ͨ�Ŷ�Ҫ����Calico/Flanneld����Ҫ�ȵ��������������ſ���ȷ�ϡ�

```shell
$ kubectl get nodes
NAME             STATUS   ROLES    AGE     VERSION
k8s01-master01   Ready    master   22h     v1.13.3
k8s01-master02   Ready    master   6h47m   v1.13.3
k8s01-node01     Ready    <none>   22h     v1.13.3
k8s01-node02     Ready    <none>   22h     v1.13.3
k8s01-node03     Ready    <none>   22h     v1.13.3

$ kubectl get pods -n kube-system
NAME                                     READY   STATUS    RESTARTS   AGE
calico-node-bt2gq                        2/2     Running   0          22h
calico-node-gln4j                        2/2     Running   0          22h
calico-node-s4xj7                        2/2     Running   0          6h47m
calico-node-ttj6q                        2/2     Running   0          22h
calico-node-wsc7s                        2/2     Running   0          22h
coredns-86c58d9df4-n8tvw                 1/1     Running   0          22h
coredns-86c58d9df4-qlq2n                 1/1     Running   0          22h
kube-apiserver-k8s01-master01            1/1     Running   0          22h
kube-apiserver-k8s01-master02            1/1     Running   0          6h47m
kube-controller-manager-k8s01-master01   1/1     Running   0          22h
kube-controller-manager-k8s01-master02   1/1     Running   0          6h47m
kube-proxy-89f5j                         1/1     Running   0          6h47m
kube-proxy-fnc9t                         1/1     Running   0          22h
kube-proxy-gf4xt                         1/1     Running   0          22h
kube-proxy-j7ltr                         1/1     Running   0          22h
kube-proxy-z4tbz                         1/1     Running   0          22h
kube-scheduler-k8s01-master01            1/1     Running   0          22h
kube-scheduler-k8s01-master02            1/1     Running   0          6h47m
```



### 11. ����Dashboard

- ��¶�˿ڣ�Node Port��30000

```yaml
$ cat <<EOF > kubernetes-dashboard.yaml
# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Configuration to deploy release version of the Dashboard UI compatible with
# Kubernetes 1.8.
#
# Example usage: kubectl create -f <this_file>

# ------------------- Dashboard Secret ------------------- #

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kube-system
type: Opaque

---
# ------------------- Dashboard Service Account ------------------- #

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Role & Role Binding ------------------- #

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
rules:
  # Allow Dashboard to create 'kubernetes-dashboard-key-holder' secret.
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["create"]
  # Allow Dashboard to create 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
- apiGroups: [""]
  resources: ["secrets"]
  resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs"]
  verbs: ["get", "update", "delete"]
  # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
- apiGroups: [""]
  resources: ["configmaps"]
  resourceNames: ["kubernetes-dashboard-settings"]
  verbs: ["get", "update"]
  # Allow Dashboard to get metrics from heapster.
- apiGroups: [""]
  resources: ["services"]
  resourceNames: ["heapster"]
  verbs: ["proxy"]
- apiGroups: [""]
  resources: ["services/proxy"]
  resourceNames: ["heapster", "http:heapster:", "https:heapster:"]
  verbs: ["get"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: kubernetes-dashboard-minimal
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard-minimal
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system

---
# ------------------- Dashboard Deployment ------------------- #

kind: Deployment
apiVersion: apps/v1beta2
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        #image: harbor.shinezone.com/shinezonetest/kubernetes-dashboard-amd64:v1.8.3
        image: k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.0
        ports:
        - containerPort: 8443
          protocol: TCP
        args:
          - --auto-generate-certificates
          # Uncomment the following line to manually specify Kubernetes API server Host
          # If not specified, Dashboard will attempt to auto discover the API server and connect
          # to it. Uncomment only if the default does not work.
          # - --apiserver-host=http://my-address:port
          #- --authentication-mode=basic
        volumeMounts:
        - name: kubernetes-dashboard-certs
          mountPath: /certs
          # Create on-disk volume to store exec logs
        - mountPath: /tmp
          name: tmp-volume
        livenessProbe:
          httpGet:
            scheme: HTTPS
            path: /
            port: 8443
          initialDelaySeconds: 30
          timeoutSeconds: 30
      volumes:
      - name: kubernetes-dashboard-certs
        secret:
          secretName: kubernetes-dashboard-certs
      - name: tmp-volume
        emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

---
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
#    kubernetes.io/cluster-service: "true"
#    addonmanager.kubernetes.io/mode: Reconcile
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30000
  selector:
    k8s-app: kubernetes-dashboard

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system

---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
EOF
```

```shell
$ kubectl create -f kubernetes-dashboard.yaml
$ kubectl create clusterrolebinding  login-on-dashboard-with-cluster-admin --clusterrole=cluster-admin --user=admin
```

 #### 11.1 ��½����

`https://172.16.1.49:30000/ `

��ȡtoken,ͨ�����Ƶ�½

```shell
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```



### 12 ����Traefik Ingress

> Traefik ʹ��DS��ʽ���в���

```yaml
$ cat <<EOF > traefik-rbac.yaml
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: default
EOF
```

```yaml
$ cat <<EOF > traefik-ui.yaml
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: default
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: default
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: k8s-traefik.shinezone.com
    http:
      paths:
      - backend:
          serviceName: traefik-web-ui
          servicePort: 80
EOF
```

```yaml
$ cat <<EOF > traefik-ds.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: default
---
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: default
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      hostNetwork: true
      restartPolicy: Always
      volumes:
      - name: ssl
        secret:
          secretName: traefik-cert
      - name: config
        configMap:
          name: traefik-conf
      containers:
      - image: traefik
        name: traefik-ingress-lb
        volumeMounts:
        - mountPath: "/etc/kubernetes/ssl"  #֤������Ŀ¼
          name: "ssl"
        - mountPath: "/root/k8s-online/traefik"        #traefik.toml�ļ�����Ŀ¼
          name: "config"
        resources:
          limits:
            cpu: 200m
            memory: 30Mi
          requests:
            cpu: 100m
            memory: 20Mi
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
        - name: https
          containerPort: 443
          hostPort: 443
        securityContext:
          privileged: true
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
        - --configfile=/root/k8s-online/traefik/traefik.toml
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: default
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      port: 80
      name: http
    - protocol: TCP
      port: 443
      name: https
    - protocol: TCP
      port: 8080
      name: admin
  type: NodePort
EOF
```

#### 12.1 ����Tracefik HTTPS

```yaml
$ cat <<EOF > traefik.toml
defaultEntryPoints = ["http", "https"]

[entryPoints]
  [entryPoints.http]
  address = ":80"
    [entryPoints.http.redirect]
    entryPoint = "https"
  [entryPoints.https]
  address = ":443"
    [entryPoints.https.tls]
      [[entryPoints.https.tls.certificates]]
      certFile = "/etc/kubernetes/ssl/shinezone.com.crt"
      keyFile = "/etc/kubernetes/ssl/shinezone.com.key"
EOF
```

```shell
$ kubectl create configmap traefik-conf --from-file=traefik.toml    //���������ֵ�
$ kubectl create secret generic traefik-cert --from-file=/etc/kubernetes/ssl/shinezone.com.key --from-file=/etc/kubernetes/ssl/shinezone.com.crt                    //���ɱ����ֵ�
```

#### 12.2 ����Traefik

```shell
$ kubectl create -f traefik-rbac.yaml 
$ kubectl create -f traefik-ds.yaml
$ kubectl create -f traefik-ui.yaml
$ kubectl get pods --all-namespaces |grep traefik
default       traefik-ingress-controller-gwcjk         1/1     Running   0         
default       traefik-ingress-controller-rphgz         1/1     Running   0         
default       traefik-ingress-controller-sd5l7         1/1     Running   0
$ kubectl get svc --all-namespaces |grep traefik
default       traefik-ingress-service   NodePort    10.105.248.48    <none>        80:30899/TCP,443:30942/TCP,8080:30686/TCP  
default       traefik-web-ui            ClusterIP   10.109.104.153   <none>        80/TCP                                    
```

#### 12.3 ����Traefik

> Ĭ�϶˿�Node IP+ NodePort,Ҳ����ʹ��Traefik������� ʹ����������

- NodePort��http://ip:node_port/dashboard/
- Traefik��https://k8s-traefik.shinezone.com  #�Ƽ�ʹ��traefik���õ�������ʽ





### 13. Prometheus-Operator

- [Prometheus Operator����](https://github.com/yanghongfei/Kubernetes#153-%E5%AE%89%E8%A3%85promethues-operator)
- ������https://github.com/coreos/prometheus-operator

- �ο��ĵ���http://www.servicemesher.com/blog/prometheus-operator-manual/

#### 13.1 ���ٿ�ʼ

> �ٷ��������ļ�������һ�������ҷ�����

```
cd contrib/kube-prometheus/manifests/
mkdir -p operator node-exporter alertmanager grafana kube-state-metrics prometheus serviceMonitor adapter
mv *-serviceMonitor* serviceMonitor/
mv 0prometheus-operator* operator/
mv grafana-* grafana/
mv kube-state-metrics-* kube-state-metrics/
mv alertmanager-* alertmanager/
mv node-exporter-* node-exporter/
mv prometheus-adapter* adapter/
mv prometheus-* prometheus/
$ ll
total 40
drwxr-xr-x 9 root root 4096 Jan  6 14:19 ./
drwxr-xr-x 9 root root 4096 Jan  6 14:15 ../
-rw-r--r-- 1 root root   60 Jan  6 14:15 00namespace-namespace.yaml
drwxr-xr-x 3 root root 4096 Jan  6 14:19 adapter/
drwxr-xr-x 3 root root 4096 Jan  6 14:19 alertmanager/
drwxr-xr-x 2 root root 4096 Jan  6 14:17 grafana/
drwxr-xr-x 2 root root 4096 Jan  6 14:17 kube-state-metrics/
drwxr-xr-x 2 root root 4096 Jan  6 14:18 node-exporter/
drwxr-xr-x 2 root root 4096 Jan  6 14:17 operator/
drwxr-xr-x 2 root root 4096 Jan  6 14:19 prometheus/
drwxr-xr-x 2 root root 4096 Jan  6 14:17 serviceMonitor/
```

- ����opertor

> �ȴ���ns��operator��quay.io�ֿ���ȡ��������ʹ�ýű���ȡ����������Ҳ��������ȥ����������apply֮ǰ��������һ����docker������ȡ��ֻ��������

```shell
kubectl apply -f .
curl -s https://zhangguanzhang.github.io/bash/pull.sh | bash -s -- quay.io/coreos/prometheus-operator:v0.26.0
kubectl apply -f operator/
# ȷ��״̬��������������ִ�У����ﾵ����quay.io�ֿ�Ŀ��ܻ�������ĵȴ����������޸ĳ�����ȡ���ġ�
$ kubectl -n monitoring get pod
NAME                                   READY     STATUS    RESTARTS   AGE
prometheus-operator-56954c76b5-qm9ww   1/1       Running   0          24s
```

- ���ű����ʲ����������ǽű�����

```shell
#!/bin/bash
[ -z "$set_e" ] && set -e

[ -z "$1" ] && { echo '$1 is not set';exit 2; }

if [ "$1" == search ];then
    shift
    which jq &> /dev/null || { echo 'search needs jq, please install the jq';exit 2; }
    img=${1%/}
    [[ $img == *:* ]] && img_name=${img/://} || img_name=$img
    if [[ "$img" =~ ^gcr.io|^k8s.gcr.io ]];then
        if [[ "$img" =~ ^k8s.gcr.io ]];then
            img_name=${img_name/k8s.gcr.io\//gcr.io/google_containers/}
        elif [[ "$img" == *google-containers* ]]; then
            img_name=${img_name/google-containers/google_containers}
        fi
        repository=gcr.io
    elif [[ "$img" =~ ^quay.io ]];then
            repository=quay.io 
    else 
        echo 'not sync the namespaces!';exit 0;
    fi
    #echo '��ѯ�õ�github,GFWԭ����ܻ�ȽϾ�,��ȷ���ܷ��ʵ�github'
    curl -sX GET https://api.github.com/repos/zhangguanzhang/${repository}/contents/$img_name?ref=develop |
        jq -r 'length as $len | if $len ==2 then .message elif $len ==12 then .name else .[].name  end'
else
    img=$1
    name=${img%:*}
    [[ $img == *:* ]] && tag=${img##*:} || tag=latest

    if [[ "$img" =~ ^gcr.io|^quay.io ]];then
        repo=zhangguanzhang/${name//\//.}
        [[ "$img" == *google-containers* ]] && repo=${repo/google-containers/google_containers}
        docker pull $repo:$tag
        docker tag $repo:$tag $img
        docker rmi $repo:$tag

    elif [[ "$img" =~ ^k8s.gcr.io ]];then
        repo=zhangguanzhang/${name/k8s.gcr.io\//gcr.io.google_containers.}
        docker pull $repo:$tag
        docker tag $repo:$tag $img
        docker rmi $repo:$tag
    else
        echo 'not sync the namespaces!';exit 0;
    fi
fi

```



- ���Ų�������CRD

```shell
# ��������ԭ�򣬿�����Ҫ�ܳ�ʱ��
kubectl apply -f adapter/
kubectl apply -f alertmanager/
kubectl apply -f node-exporter/
kubectl apply -f kube-state-metrics/
kubectl apply -f grafana/
kubectl apply -f prometheus/
kubectl apply -f serviceMonitor/

#�鿴����
kubectl -n monitoring get all
```





#### 13.5 ʹ��Traefik �������Prometheus Grafana 

> �������ļ�����apply����
>
> kubectl apply -f  traefik-grafana.yaml 
>
> kubectl apply -f  traefik-prometheus.yaml 

- `traefik-grafana.yaml `

```yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
spec:
  rules:
  - host: k8s-grafana.shinezone.com
    http:
      paths:
      - path: /
        backend:
          serviceName: grafana
          servicePort: 3000
```

- ` traefik-prometheus.yaml `

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus-k8s
  namespace: monitoring
spec:
  rules:
  - host: k8s-prometheus.shinezone.com
    http:
      paths:
      - path: /
        backend:
          serviceName: prometheus-k8s
          servicePort: 9090
```

- Ч��ͼ

![](./images/grafana-cluster.png)



![](./images/grafana-nodes.png)



### ά�����

#### 01. ǿ��ɾ��Pod

```shell
$ kubectl delete  pods calico-node-wkmfl   --grace-period=0 --force --namespace kube-system
##ǿ������pod
kubectl get pods coredns-bb849946b-jflvr -n kube-system -o yaml | kubectl replace --force -f -
```

#### 02. ���»�ȡ��ȺToken

> ��Ⱥ��ʼ��ʱ��δ����tokenTTL: "0s" ��ôĬ�����ɵ�token����Ч��Ϊ24Сʱ��������֮�󣬸�token�Ͳ������ˡ��������ʧkubeadm join���뼯Ⱥ���Ҳͬ������ͨ�����·������

```shell
#��������token
$ kubeadm token create

#�鿴
$ kubeadm token list

#��ȡca֤��sha256����hashֵ
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

#ʹ�������������ɵ�token��hashֵ�������ڵ���뼯Ⱥ
$ kubeadm join --token qfk0zc.uobckpxnh54v5at3 --discovery-token-ca-cert-hash sha256:68efb4e280d110f3004a4e16ed09c5ded6c2421cb24a6077cf6171d5167b04d2  10.0.0.15:6443 --skip-preflight-checks
```

#### 03. Master�ڵ�Ҳ����Pod����

> Ĭ�ϼ�Ⱥ��װ��ɺ�Master�ڵ��ǲ�����Pod����ģ���Դ����/ʵ�黷������Master �ڵ�Ҳ�������pod

```shell
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```



#### 04. modprobe: FATAL: Module br_netfilter not found

> ��Щϵͳ�ں�����û��br_netfilter����ʱ����Ҫ���Ǹ���ϵͳ�ںˡ�

```shell
modprobe br_netfilter   //����: modprobe: FATAL: Module br_netfilter not found
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
EOF
sysctl -p /etc/sysctl.d/k8s.conf
ls /proc/sys/net/bridge
```

- ����취��
  - �ο��ĵ���https://linux.cn/article-8310-1.html

```shell
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
yum --enablerepo=elrepo-kernel install kernel-ml -y
# �޸� GRUB ��ʼ��ҳ��ĵ�һ���ں˽���ΪĬ���ں�
sed -i s#GRUB_DEFAULT=saved#GRUB_DEFAULT=0#g /etc/default/grub  
# ���´����ں�����
grub2-mkconfig -o /boot/grub2/grub.cfg
# �����������ں�
reboot
```



### �ο��ĵ�

����ο��ĵ���

https://www.kubernetes.org.cn/4956.html

https://www.qikqiak.com/post/how-to-use-ipvs-in-kubernetes/

http://www.servicemesher.com/blog/prometheus-operator-manual/

https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/

https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta1

https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/ipvs/README.md