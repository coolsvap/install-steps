Replace <<KUBE-MASTER-IP>>, <<KUBE-MINION-1-IP>>, <<KUBE-MINION-2-IP>>, <<KUBE-MINION-3-IP>> with appropriate hostname or IP address. Only /etc/hosts configuration needs a change if hostname is used.

Network-Setup

Kube-Master : <<KUBE-MASTER-IP>>
Kube-Minion-1 : <<KUBE-MINION-1-IP>>
Kube-Minion-2 : <<KUBE-MINION-2-IP>>
Kube-Minion-3 : <<KUBE-MINION-3-IP>>


On All Nodes
============
systemctl stop firewalld
systemctl disable firewalld

yum erase docker-engine -y

yum -y install ntp
systemctl start ntpd
systemctl enable ntpd

vi /etc/hosts
kube-master <<KUBE-MASTER-IP>>
kube-minion-1 <<KUBE-MINION-1-IP>>
kube-minion-2 <<KUBE-MINION-2-IP>>
kube-minion-3 <<KUBE-MINION-3-IP>>

On Master
==========
yum -y install etcd kubernetes

vi /etc/etcd/etcd.conf

ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://localhost:2379"

vi /etc/kubernetes/apiserver

KUBE_API_ADDRESS="--address=0.0.0.0"
KUBE_API_PORT="--port=8080"
KUBELET_PORT="--kubelet_port=10250"
KUBE_ETCD_SERVERS="--etcd_servers=http://127.0.0.1:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=10.254.0.0/16"
KUBE_ADMISSION_CONTROL="--admission_control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ResourceQuota"
KUBE_API_ARGS=""


for SERVICES in etcd kube-apiserver kube-controller-manager kube-scheduler; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES 
done

etcdctl mk /atomic.io/network/config '{"Network":"172.17.0.0/16"}'

kubectl get nodes

On Minions
===========
yum -y install flannel kubernetes

vi /etc/sysconfig/flanneld
FLANNEL_ETCD="http://<<KUBE-MASTER-IP>>:2379"

vi /etc/kubernetes/config
KUBE_MASTER="--master=http://<<KUBE-MASTER-IP>>:8080"


on Minion 1
============
vi /etc/kubernetes/kubelet
 
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
# change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=<<KUBE-MINION-1-IP>>"
KUBELET_API_SERVER="--api_servers=http://<<KUBE-MASTER-IP>>:8080"
KUBELET_ARGS=""


on Minion 2
============
vi /etc/kubernetes/kubelet
 
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
# change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=<<KUBE-MINION-2-IP>>"
KUBELET_API_SERVER="--api_servers=http://<<KUBE-MASTER-IP>>:8080"
KUBELET_ARGS=""

on Minion 3
============
vi /etc/kubernetes/kubelet
 
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
# change the hostname to this host’s IP address
KUBELET_HOSTNAME="--hostname_override=<<KUBE-MINION-3-IP>>"
KUBELET_API_SERVER="--api_servers=http://<<KUBE-MASTER-IP>>:8080"
KUBELET_ARGS=""

On All Minions
==============

for SERVICES in kube-proxy kubelet docker flanneld; do
    systemctl restart $SERVICES
    systemctl enable $SERVICES
    systemctl status $SERVICES 
done

ip a | grep flannel | grep inet


On All Nodes
============
systemctl start docker && systemctl enable docker && systemctl status docker
