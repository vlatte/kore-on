- [Features](#features)
- [Supported Components](#supported-components)
- [Required packages](#required-packages)
- [Deploy kubernetes HA cluster using koreon](#deploy-kubernetes-ha-cluster-using-koreon)
- [Deploy kubernetes All-in-one cluster for testing](#deploy-kubernetes-all-in-one-cluster-for-testing)

Koreon은 On-premise 환경에서 Kubernetes cluster의 생성, worker node 추가/삭제, 클러스터 업그레이드, 클러스터 삭제 기능을 제공한다.
또한, Harbor registry와 NFS server를 함께 설치함으로 운영환경에서 즉시 사용할 수 있으며, 폐쇄망(Air-gap) 환경을 위해 압축파일(Harbor, system package)로 클러스터를 설치할 수 있는 기능을 제공한다.
대상 서버의 정보(IP, SSH key)등을 `koreon.toml`파일에 기술하고 간단히 `koreonctl create` 명령을 실행하면 간단히 설치된다.

## Features
- Deploys Single or Highly Available (HA) Kubernetes
- Upgrade Kubernetes cluster
- Add/Delete worker node
- Install harbor registry
- Install NFS server
- Air-Gap installation

## Supported Components

- Core
  - [kubernetes](https://github.com/kubernetes/kubernetes/tree/master/CHANGELOG) v1.19.10-v1.19.12, v1.20.6-v1.20.8, v1.21.0-v1.21.2
  - [etcd](https://github.com/etcd-io/etcd/releases) v3.4.16
  - [docker-compose](https://github.com/docker/compose/releases) v1.29.2  
  - [docker](https://www.docker.com/) v19.03.15
  - [containerd](https://containerd.io/) v1.4.3
  - [crictl](https://github.com/kubernetes-sigs/cri-tools) v1.19.0, v1.20.0, v1.21.0
  
- Network Plugin
  - [calico](https://github.com/projectcalico/calico/releases) v3.19.1
  
- Application
  - [coredns](https://github.com/coredns/coredns) v1.7.0, v1.8.0
  - [haproxy](https://hub.docker.com/_/haproxy?tab=tags&page=1&ordering=last_updated) v2.4.2  
  - [nfs-subdir-external-provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner/releases) v4.0.2  
  
- Registry
  - [harbor](https://github.com/goharbor/harbor/releases) v2.3.0

## Required packages
 * docker runtime

## Deploy kubernetes HA cluster using koreon

* Koreon 명령어

|CMD           |설명                                             |
| ------------ | ---------------------------------------------- |
|version       |버전정보 조회|
|init          |koreon.toml 다운로드|
|create        |클러스터 생성|
|destroy       |클러스터 제거|
|apply         |클러스터의 워크노드 개수 조정 또는 쿠버네티스 버전 업그레이드|
|prepare-airgap|폐쇄망에 사용할 리소스 및 private registry tgz 파일 생성|


  
1. Make clean working directory.
```bash
$ mkdir /tmp/koreon; cd /tmp/koreon
```

2. Generate ssh key and copy it to target servers
```bash
$ ssh-keygen -t rsa -N '' -f ./id_rsa
$ ssh-copy-id ./id_rsa root@192.168.77.190
$ ssh-copy-id ./id_rsa root@192.168.77.191
$ ssh-copy-id ./id_rsa root@192.168.77.192
$ ssh-copy-id ./id_rsa root@192.168.77.193
$ ssh-copy-id ./id_rsa root@192.168.77.194
```
3. Edit koreon.toml  
   
```toml
[koreon]
cluster-name = "koreon-ha"
#cert-validity-days = 3650

#closed-network = true
#local-repository = "http://192.168.88.145:8080"
#local-repository-archive-file = "/tmp/koreon/local-repo.20220224_071700.tgz"
#install-dir = "/var/lib/koreon"
#debug-mode = true

[kubernetes]
version = "1.20.6"
#container-runtime = "containerd"
#kube-proxy-mode = "ipvs"
#vxlan-mode = true
#service-cidr ="10.96.0.0/12"
#pod-cidr="10.32.0.0/12"
#node-port-range="30000-32767"
audit-log-enable = true
#api-sans = ["192.168.1.9"]


[kubernetes.etcd]
ip = ["192.168.88.91","192.168.88.92","192.168.88.93"]
private-ip = ["172.33.88.91","172.33.88.92","172.33.88.93"]
encrypt-secret = true


[node-pool]
#data-dir = "/data"

[node-pool.security]
ssh-user-id = "ubuntu"
private-key-path = "./id_rsa"

[node-pool.master]
ip = ["192.168.88.91","192.168.88.92","192.168.88.93"]
private-ip = ["172.33.88.91","172.33.88.92","172.33.88.93"]
lb-ip = "192.168.88.91"
#isolated = true
haproxy-install = true

[node-pool.node]
ip = ["192.168.88.95"]
private-ip = ["172.33.88.95"]


[shared-storage]
install = true
storage-ip = "192.168.88.96"
volume-dir = "/data/cluster"
volume-size = 1000


[private-registry]
install = true
registry-ip = "192.168.88.96"
data-dir = "/data/harbor"
public-cert = false
#registry-archive-file = "/tmp/koreon/harbor.20220224_072307.tgz"

[private-registry.cert-file]
ssl-certificate = ""
ssl-certificate-key = ""
```
<br/>

  - Create kubernetes HA cluster using koreon

```bash
$ koreonctl create
```  
<p align="center">
  <a href="https://asciinema.org/a/YNANH4phrwtSB7x8dDUv15sQe">
  <img src="https://asciinema.org/a/YNANH4phrwtSB7x8dDUv15sQe.png" width="600">
  </a>
</p>

  - Add worker node using koreon
```bash
$ koreonctl apply
```  
<p align="center">
  <a href="https://asciinema.org/a/1Sxsfjvgh2Ap8xct8adSvgrJd">
  <img src="https://asciinema.org/a/1Sxsfjvgh2Ap8xct8adSvgrJd.png" width="600">
  </a>
</p>

  - Remove worker node using koreon
```bash
$ koreonctl apply
```   
<p align="center">
  <a href="https://asciinema.org/a/WukDpcLsPG05fw3OT0lwtmT9s">
  <img src="https://asciinema.org/a/WukDpcLsPG05fw3OT0lwtmT9s.png" width="600">
  </a>
</p>

  - Upgrade kubernetes cluster using koreon 
```bash
$ koreonctl apply
```   
<p align="center">
  <a href="https://asciinema.org/a/GiHddAIsOjAwk2clS5cSNVuE4">
  <img src="https://asciinema.org/a/GiHddAIsOjAwk2clS5cSNVuE4.png" width="600">
  </a>
</p>

  - Destroy kubernetes cluster using koreon 
```bash
$ koreonctl destroy
```   
<p align="center">
  <a href="https://asciinema.org/a/bOvqRrZFFGk27c2p7prcaUTmK">
  <img src="https://asciinema.org/a/bOvqRrZFFGk27c2p7prcaUTmK.png" width="600">
  </a>
</p>

## Deploy kubernetes All-in-one cluster for testing

  - Create kubernetes cluster
```bash
$ koreonctl create
```    
<p align="center">
  <a href="https://asciinema.org/a/oIYXztzPi03m7OFO98VPxPtLs">
  <img src="https://asciinema.org/a/oIYXztzPi03m7OFO98VPxPtLs.png" width="600">
  </a>
</p>

  - Upgrade kubernetes cluster 
```bash
$ koreonctl apply
```    
<p align="center">
  <a href="https://asciinema.org/a/GFCrrp92aYqY4DhuhOvwG0p9b">
  <img src="https://asciinema.org/a/GFCrrp92aYqY4DhuhOvwG0p9b.png" width="600">
  </a>
</p>
