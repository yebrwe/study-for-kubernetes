# 기본 설정(도커&쿠버네티스 설치)

## docker install

```bash
sudo apt-get update
sudo apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io

sudo mkdir /etc/docker
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF

sudo systemctl enable docker
sudo systemctl daemon-reload
sudo systemctl restart docker
docker version
```

## kubeadm install

```bash
# Swap disabled. You MUST disable
swapoff -a && sed -i '/swap/s/^/#/' /etc/fstab

# Letting iptables see bridged traffic
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## master init

```bash
kubeadm init
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
#token 저장
kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
kubectl get nodes

#root 일경우
export KUBECONFIG=/etc/kubernetes/admin.conf
```

## 쿠버네티스 자동완성

```bash
#kubectl completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc

#kubeadm completion
source <(kubeadm completion bash)
echo "source <(kubeadm completion bash)" >> ~/.bashrc
```

## POD ~= dokcer container?

## 쿠버네티스 POD 생성 - nginx

```bash
kubectl run webserver --image:nginx:1.14 --port 80
```

## 쿠버네티스 앱 배포(POD 생성이랑 유사)

```bash
kubectl create deployment --image:httpd --replicas=3
```

## 콘솔에서 웹페이지 보는 프로그램

```bash
apt-get update
apt-get install -y elinks
elinks [ip address]
```

## POD 내부 접속

```bash
kubectl exec webserver -it -- /bin/bash
```

## POD 수정[vi mode]

```bash
kubectl edit deployment.apps mainui
```

## POD yaml 생성

```bash
kubectl run webserver --image=nginx:1.14 --port 80 --dry-run -o yaml > webserver-pod.yaml
```

## POD yaml 실행

```bash
kubectl create -f webserver-pod.yaml
```

## POD 삭제

```bash
kubectl delete pod webserver
```

# 네임스페이스 확인

```bash
kubectl get namespaces
```

# 네임스페이스 생성

```bash
kubectl create namespace blue
```

# 네임스페이스 yaml 생성

```bash
kubectl create namespace blue --dry-run -o yaml > blue-ns.yaml
```

# 특정 네임스페이스의 POD 조회

```bash
kubectl get pods --namespace kube-system
```

# 쿠버네티스 설정 확인

```bash
kubectl config view
```

# 쿠버네티스 컨텍스트 등록

```bash
kubectl config set-context blue@kubernetes --cluster=kubernetes --user=kubernetes-admin --namespace=blue
```

# 쿠버네티스 현재 컨텍스트 확인

```bash
kubectl config current-context
```

# 쿠버네티스 컨텍스트 변경

```bash
kubectl config use-context blue@kubernetes
```

# 쿠버네티스 API 버전 확인

```bash
kubectl explain pod
```
