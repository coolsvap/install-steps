fdisk /dev/sdb
fdisk -l
mkdir /data
mkfs.btrfs /dev/sdb1
service docker stop
ls  /var/lib/docker/
du -h  /var/lib/docker/
rm -rf  /var/lib/docker/*
vi /etc/fstab
	/dev/sdb1 /var/lib/docker btrfs defaults 0 1
mount -a
fdisk -l
df -h

vi /etc/docker/daemon.json

{
  "storage-driver": "btrfs"
}


service docker start
docker info
docker info |grep Storage


service docker stop
vi /etc/sysconfig/docker

INSECURE_REGISTRY="--insecure-registry registry:5000"


vi /etc/hosts

ping registry
vi  /etc/systemd/system/docker.service

[Service]
MountFlags=shared
EnvironmentFile=/etc/sysconfig/docker
ExecStart=/usr/bin/docker daemon $INSECURE_REGISTRY

systemctl daemon-reload && systemctl restart docker
docker info

docker volume create registry-backup

docker run -d -p 5000:5000 --restart=always --name registry   -v registry-backup:/var/lib/registry   registry

