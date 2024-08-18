# k8s-study
k8s 실습 repo

## VM 3개 
1. control plane
2. node 1
3. node 2
   
### 설치전 환경설정

```
# docker 설치

# 도커 however docker-cli:arm-64 ~~~ 오류
curl -fsSL get.docker.com | CHANNEL=stable sh
sudo apt install docker-ce
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

```
# swap 끄기
1) Swap disabled
# swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab

2) Letting iptables see bridged traffic
# cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# sysctl --system

# 방화벽 disable
3) Disable firewall
# ufw disable
# systemctl stop ufw

```

### 포트 열려있나 확인
```
nc 127.0.0.1 6443 -v
```
### kubernetes download
```
sudo apt-get update
# apt-transport-https may be a dummy package; if so, you can skip that package
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
### kubernetes 공개 저장소의 공개 서명 키 다운로드

```
# If the directory `/etc/apt/keyrings` does not exist, it should be created before the curl command, read the note below.
# sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### apt 저장소 추가
```
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```





### kubelet, kubeadm, kubectl download
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

systemctl start kubelet
systemctl enable kubelet

```

## master plane 구성

```
kubeadmn init

# 완료 시
kubeadm join 10.100.0.104:6443 --token 1ou05o.kkist9u6fbc2uhp3 --discovery-token-ca-cert-hash sha256:8d9a7308ea6ff73.........576c112f326690
전체 구문 복사 후
원하는 파일에 저장 ex) vi token.txt

# 디렉토리 만들고 config 파일 복사

mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### CNI 설정 (Container Network Interface)

```
# weave cni download(다른 CNI 설치해도됨)
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```

### 확인
```
kubectl get nodes
```

### 만약 status 상태가 not ready 로 계속 나올 시

https://velog.io/@juhoon212/Kubernetes-Status-Not-Ready [참고]

## worker node 구성

### ERROR

```
# error execution phase preflight: couldn't validate the identity of the API Server: invalid discovery token CA certificate hash: invalid hash "sha256:807b108710efaa0eebeb4dfb19e481a3fa", expected a 32 byte SHA-256 hash, found 17 bytes To see the stack trace of this error execute with --v=5 or higher

kubeadm token list
kubeadm token delete [token이름]
kubeadm token create --print-join-command
```
