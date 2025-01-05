
### Introduction to Deployment with Kubeadm

#### kubeadm

쿠버네티스 클러스터는 kubeAPI서버, etcd, controller 등 다양한 컴포넌트로 구성된다.
그리고 이러한 컴포넌트 간의 통신을 가능하게 하기 위해서 보안과 인증서 관련한 몇 가지 요구사항들도 필요하다.
다양한 컴포넌트를 각각의 노드에서 개별적으로 모두 설치하고 configuration file들을 수정하여 컴포넌트 간의 통신을 확인하고 인증서가 잘 작동하는지 확인하는 것은 지루하고 복잡한 작업이다.
-> *kubeadm* 툴은 이러한 모든 작업을 처리해준다.

#### Steps
![[Pasted image 20240707214534.png|500]]
kubeadm 툴을 사용해서 쿠버네티스 클러스터를 설정하는 단계

1. 클러스터를 구성하기 위한 다중 시스템 또는 프로비저닝된 가상머신이 있어야 한다. 
    -> 시스템이 생성되면 하나의 노드를 마스터로 지정하고 다른 노드는 워커노드로 둔다.
2. 호스트에 컨테이너 런타임을 설치 -> *containerd*
3. kubeadm  모든 노드에 설치 
   kubeadm은 필요한 모든 컴포넌트를 올바른 노드에 올바른 순서로 설치하고 구성하는 쿠버네티스 솔루션 부트스트랩을 지원한다.
4. 마스터노드 initialize
5. Pod Network
   마스터 서버에 필요한 모든 컴포넌트가 설치되고 구성되고, 마스터가 초기화되면, 
   워커 노드를 마스터에 결합하기 전에 네트워크 전제조건이 충족되었는지 확인해야 한다. 
   쿠버네티스에는 마스터 노드와 워커 노드 사이의 특별한 네트워킹 솔루션이 필요한데, 이를 파드 네트워크라고 한다.
6. Join Node
   파드 네트워크가 셋팅되면 마지막 단계로는 워커 노드를 마스터 노드에 조인시킨다.

이렇게 쿠버네티스에서 우리의 애플리케이션을 시작하기 위한 준비가 완료된것이다.

#### Installing kubeadm - Document
https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

#### Deploy with Kubeadm
https://github.com/kodekloudhub/certified-kubernetes-administrator-course?tab=readme-ov-file
virtual box & vagrant 설치

https://github.com/kodekloudhub/certified-kubernetes-administrator-course/blob/master/docs/11-Install-Kubernetes-the-kubeadm-way/04-Demo-Deployment-with-Kubeadm.md
```
vagrant up
vagrant status

vagrant ssh kubemaster
vagrant ssh kubenode01
vagrant ssh kubenode02
```

Check you can SSH to each VM
![[Pasted image 20240707215626.png|400]]

#### Initialize the nodes.
- Configure kernel parameters
  https://v1-29.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/#forwarding-ipv4-and-letting-iptables-see-bridged-traffic
  -> document
  -> 각각 노드마다 실행.
```
lsmod | grep br_netfilter
lsmod | grep overlay 

sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
  ```
  
- Install `containerd` container driver and associated tooling
  -> document - https://v1-29.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd
  -> github - https://github.com/containerd/containerd/blob/main/docs/getting-started.md
  -> install docker engine in linux (docker docs)
  -> cgroup -> https://v1-29.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroup-drivers
  
- Install Kubernetes software
  -> install kubeadm, kubelet, kubectl
  -> https://v1-29.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

#### Initialize controlplane node
  -> https://v1-29.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
-> https://v1-29.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#initializing-your-control-plane-node
in master node
```
ip add
-> interface ip -> apiserver


sudo kubeadm init \
   --apiserver-cert-extra-sans=kubemaster01 \
   --apiserver-advertise-address 192.168.56.2 \
   --pod-network-cidr=10.244.0.0/16


# Set up the default kubeconfig file
{
mkdir ~/.kube
sudo cp /etc/kubernetes/admin.conf ~/.kube/config
sudo chown vagrant:vagrant ~/.kube/config
}
 # -> 클러스터에 연결하기 위해
 kubectl get pod 
 로 클러스터에 정상 연결되었는지 확인.
 
```

#### Initialize the worker nodes
- installing addon document -> https://k8s.aluopy.cn/docs/concepts/cluster-administration/addons/
  -> weave net (지금 제공하고있지 않음.)
  in master node
```
kubectl apply -f https~~ weave-demonset-k8s.yaml
```

- *kubeadm join*
  in worker node,
```
sudo kubeadm join 192.168.56.2:6443 --token ~~ --discovery-token-ca-cert-hash ~~
```


#### test
Install the kubeadm and kubelet packages on the controlplane and node01 nodes.(Use the exact version of 1.27.0-2.1 for both.)

컨트롤플레인과 node01 노드에 kubeadm 및 kubelet 패키지를 설치

[https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)
```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

어떤 release 사용하고 있는지 check
![[Pasted image 20240707223939.png|500]]


```shell
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

sudo mkdir -m 755 /etc/apt/keyrings

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.27/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.27/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update

sudo apt-get install -y kubelet=1.29.0-2.1 kubeadm=1.29.0-2.1 kubectl=1.29.0-2.1

sudo apt-mark hold kubelet kubeadm kubectl


```


2.How many nodes are part of kubernetes cluster currently?
Are you able to run `kubectl get nodes`?

![[Pasted image 20240707224852.png]]

Lets now bootstrap a `kubernetes` cluster using `kubeadm`.
The latest version of Kubernetes will be installed.

in master node,
```
kubeadm init --apiserver-cert-extra-sans=controlplane --apiserver-advertise-address=192.20.76.12 --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
![[Pasted image 20240707225143.png|500]]

in node01,
```
kubeadm join 192.20.76.12:6443 --token bt6522.n7zqn0fpw4melbvx --discovery-token-ca-cert-hash sha256:eae5e28ea639c53cf118dc5afa0de134320fcc43ebd2e1ea090ade42004f0b39
```

https://kubernetes.io/docs/concepts/cluster-administration/addons/
-> flannel
https://github.com/flannel-io/flannel#deploying-flannel-manually
in master node,
```
kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```
