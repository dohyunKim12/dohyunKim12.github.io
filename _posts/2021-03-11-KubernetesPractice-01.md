---
layout: post
title: "Kubernetes 실습(1)"
summary: Docker & Kubernetes 배경지식 학습 및 환경 setting.
author: Dohyun Kim
date: 2021-03-11 21:30:00 -0400
category: Kubernetes
thumbnail: /assets/img/posts/Kubernetes.png
comments: true
---

#### Summary
3.11 을 기점으로 4월 중순까지 주1회 Kubernetes 실습을 진행한다.

이번 주차에는 간단히 ```Cgroup```과 ```Docker```에 대해 알아보고 Kubernetes 실습을 위한 환경을 구성해 본다.

---

#### Cgroup(Control Group)
process들의 자원 사용을 제한하고 격리시키는 Linux kernel의 기능이다.

![image](https://user-images.githubusercontent.com/72643027/111981874-9e534f00-8b4b-11eb-841b-8bf14840b63c.png){: width="70%" height="70%"}

#### Docker & Container
**Docker & Container**는 이 위에서 동작하는 단일 Linux Kernel을 사용하는 제어 Host가,  
다수의 분리된 Linux system 을 동작시키기 위한 가상화 기법이다.  
Docker는 Linux 응용 program들을 container 안에 배치시키는 것을 **자동화** 하는 Container 관리 SW이다.

#### Container Orchestration
컨테이너의 **배포, 관리, 확장, networking, 가용성**을 자동화해주는 도구이다.  
기능 : 
- provisioning 및 배포
- 설정 및 scheduling
- resource 할당
- Container 가용성 확보
- Load balancing 및 Traffic routing
- Container 상태 Monitoring
- Container 간 상호 작용의 보안 유지   

ex) Kubernetes, Docker Swarm, Apache Mesos

#### Object
- **POD** : Kubernetes 의 가장 기본적인 배포 단위로, Container를 포함하는 단위.  
-> Kubernetes는 하나의 Container를 개별적으로 하나씩 배포하는 것이 아니라, POD라는 단위로 배포하며, POD는 하나 이상의 Container를 포함.
- **Service** : Kubernetes에서 변하지 않는 IP주소와 port를 제공.  
-> Kubernetes에서 POD는 일회성이라 언제든지 이동하거나 사라질 수 있어, POD를 연결하기 위한 단일 진입점 역할을 해 준다.
- **Volume** : Container의 외장 디스크  
-> POD가 시작할 때 Container마다 Local disk를 생성하여 시작되는데, 이러한 Local disk의 경우 Container가 종료되면 기록된 내용이 유실되어 영구적으로 내용을 저장하고자 할 때 이용.
- **Namespace** : 한 Kubernetes cluster 내의 논리적인 분리 단위.

--- 

### Docker & Kubernetes 설치 및 환경구축.
다음 내용들은 회사의 vdi (CentOS7) 가상 머신 두 대를 이용하였음.  
(한 대는 Master node, 다른 한 대는 Worker node 로 구성할 것임.)  
(root 계정으로 접속을 가정.)

![image](https://user-images.githubusercontent.com/72643027/111987211-41a76280-8b52-11eb-9e62-5fcfe98e77ff.png){: width="80%" height="80%"}

이 중 277번 Machine 이 Master node, 278번 Machine이 Worker node로 설정하였음

- Docker 설치
    1. Yum update  
    ```yum update -y```
    2. Docker 설치 & enable & check  
    ```yum install -y docker```  
    ```systemctl enable docker && systemctl start docker```  
    ```docker version```
- Kubernetes Yum Repository 구성  
    ```
    sudo bash -c 'cat <<EOF > /etc/yum.repos.d/kubernetes.repo
    >[kubernetes]
    >name=Kubernetes
    >baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
    >enabled=1
    >gpgcheck=1
    >repo_gpgcheck=1
    >gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
    >exclude=kube*
    >EOF'
    ```
- 통신 문제를 방지하기 위한 SE Linux 모드 변경  
    ```
    setenforce 0
    sed -i 's/^SELINUX=enforcing$/SELINUX=premissive/' /etc/selinux/config
    ```
- Kubernetes 설치 & Kubelet 활성화
    ```
    yum install -y kubeadm-1.15.5-0.x86_64 kubectl-1.1.5.5-0.x86_64 kubelet-1.15.5-0.x86_64 --disableexcludes=kubernetes
    systemctl enable kubelet && systemctl start kubelet
    ```
- IPTables(방화벽 설정하는 도구) 설정 파일 작성
    ```
    bash -c 'cat << EOF > /etc/sysctl.d/k8s.conf
    >net.bridge.bridge-nf-call-ip6tables = 1
    >net.bridge.bridge-nf-call-iptables = 1
    >EOF'
    ```
- IPTables 설정 파일 적용  
    ```sysctl --system```

- br_netfileter 모듈 load  
    ```lsmod | grep br_netfilter```

- Port 허용
    ```
    firewall-cmd --permanent --add-port=6443/tcp
    firewall-cmd --permanent --add-port=10250/tcp
    firewall-cmd --reload
    ```
- Kubernetes 초기화에 사용될 이미지 pull  
    ```kubeadm config images pull```

- Kubernets Cluster 설정(Master Only)  
    ```kubeadm init --pod-network-cidr=10.244.0.0/16```
- Swap error처리
    ```swapoff -a```, 또는  ```vi /etc/fstab```에서 swap 설정 주석처리 -> 영구적으로 끔.

- Kubernetes Cluster 접근 설정(Master Only)
    ```
    mkdir -p $HOME/.kube
    cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    chown $(id -u):$(id -g) $HOME/.kube/config
    ```
- Kube flannel 적용(Master Only)
    ```
    wget https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
    ip a # Network Interface 명 확인
    vi Kube-flannel.yml # command에 --iface=network-interface명 추가.
    ```
- CNI(Container Network Interface) 설치(Master Only)
    ```
    curl https://docs.projectcalico.org/archive/v3.8/manifests/calico.yaml -O
    kubectl apply -f calico.yaml
    kubectl apply -f kube-flannel.yml
    ```
- Kubernetes 주요 POD 동작 확인 & master node 연결 확인(Master Only)
    ```
    kubectl get pod -n kube-system
    kubectl get nodes
    ```

![image](https://user-images.githubusercontent.com/72643027/111987508-a5319000-8b52-11eb-96f7-36c1b25da02d.png){: width="70%" height="70%"}

여기까지 성공하였다면, Master node 구성 완료.  
이제 Master node 에 Worker node를 붙여 보자.

- Master node에 추가(Worker Only)  
    ```kubeadm join [master-ip]:6443 --token [token] --discovery-token-ca-cert-hash sha256:[hash]```

- Token & Hash값 확인(Master node에서 확인)
    ```
    kubeadm token list
    openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
    openssl dgst -sha256 -hex | sed 's/^.* //'
    ```
- Token list에 값이 없을 경우 Token 생성  
    ```kubeadm token create```

결과 확인.

![image](https://user-images.githubusercontent.com/72643027/111988091-5f28fc00-8b53-11eb-9110-d0a7ab69684f.png){: width="70%" height="70%"}







