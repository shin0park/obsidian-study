## Pod Networking
![[Pasted image 20240609013403.png|500]]
kubernetes Pod Networking 요구사항
- 모든 파드는 IP 주소를 가진다.
- 모든 파드는 동일 노드내의 다른 모든 파드들과 통신할 수 있다.
- 모든 파드는 다른 노드의 모든 파드들과 통신 할 수 있다.

이러한 요구사항을 해결하는 Networking Model은 어떻게 구현할 수 있을까?
많은 네트워킹 솔루션이 이를 구현하고 있지만,
![[Pasted image 20240609013715.png|500]]
이전 강의에서 학습한 네트워킹 개념과 ip address, namespace networking을 연관시켜 이해해 볼 수도 있다. 

![[Pasted image 20240609014632.png|500]]
위의 이미지처럼,
각 노드에 컨테이너들이 생성되면, kubernetes는 컨테이너에 대한 network namespace를 생성한다. 그리고 이들간의 통신을 가능하기 위해 이러한 namespace를 네트워크에 연결해야하는데,
이전 강의에서 다뤘던, bridge network, namespace network를 통해서 이를 구현 할 수 있다.
따라서 각 노드에 bridge 를 생성하고 ip address를 할당한다.

![[Pasted image 20240609014645.png|500]]
나머지 단계는 새 컨테이너가 생성될때마다 *각 컨테이너*에 대해 수행되도록 스크립트를 작성해본다.
앞으로 각 컨테이너에 대해 여러번 실행될 수 있도록 한다.
스크립트 내용은 이전강의에서 배운것과 동일하다.
- 컨테이너를 네트워크에 연결하기 위해 veth인 가상인터페이스가 필요하다. `ip link add` 명령어를 통해 생성한다.
- `ip link set` 명령어를 통해 한쪽 끝을 컨테이너에 부착 다른 한쪽 끝을 bridge에 부착한다.
- `ip addr add` 명령어를 통해 ip 주소를 할당하고 `ip route add`를 통해 default gateway에 route를 추가한다.(같은 노드안에서의 통신이므로)
- `ip link set`명령어로 인터페이스를 bring up 한다.
이로써 동일 노드내에서 컨테이너간의 통신이 가능해지게 된다.
해당 스크립트를 다른 노드에서도 동일하게 적용하여, 각 컨테이너를 자체 내부 네트워크에 연결하도록 한다.
*따라서 kubernetes pod network의 조건인 파드들은 모두 고유한 ip를 얻으며, 동일 노드 내의 통신이 가능해진다.*

이제는 서로 다른 노드간의 파드 통신을 다뤄보자.

![[Pasted image 20240609020133.png|500]]
먼저 다른 노드에 있는 파드의 ip를 ping을 날려보면 다른 네트워크에 있기 때문에 해당 주소가 어디에 있는지 알지 못할 것이다. 
따라서, 노드의 route table에 route를 새로 추가하여 트래픽을 10.244.2.2로 라우팅 할 수 있도록 구성하여, ping을 할 수도 있다.
하지만 이런 방식은 각 route table마다 모든 경로에 대해 route를 추가해줘야 하고, 네트워크 아키텍처가 복잡해질수록 더 많은 configuration이 필요해지게 된다. 

-> 각 서버에 route를 구성하는 대신에 네트워크에 라우터가 있는 경우 해당 route table에서 모두 route를 관리하는 것이 좋은 방법이다.
![[Pasted image 20240609020142.png|500]]
따라서 위 이미지와 같이 각 노드에서의 개별 가상 네트워크를 하나로 합치는 `10.233.0.0/16` 대역의 단일 대규모 네트워크를 형성하여, 하나의 route table에서 모두 관리한다.

![[Pasted image 20240609020209.png|500]]
이와 같이 bridge network, route tabled통해서 동일내 노드간, 다른 노드간의 파드 통신들을 수행했는데, 각 컨테이너마다 해당 스크립트가 어떻게 실행되게 할 수 있을까?
수천개의 파드가 생성되는 대규모 환경에서 수동으로 우리가 스크립트를 돌릴 수는 없는 일일 것이다.
-> *CNI*
바로 CNI가 이 역할을 수행해준다.
- CNI는 컨테이너를 생성하는 즉시 스크립트를 호출하는 방법을 kubnernetes에게 알려준다.
- CNI의 기준에 맞게 스크립트를 생성해야 한다. (네트워크 컨테이너 추가하는 섹션, 네트워크에서 컨테이너 인터페이스 삭제, IP 주소 해제하는 삭세 섹션)
- 각 노드의 kubelet은 컨테이너 생성을 담당한다. - 컨테이너가 생성될때마다 kubelet은 CNI configuration과 bin디렉토리를 검색하여 스크립트를 찾은 다음 add, container name, namespace id로 스크립트를 실행한다.

## CNI

![[Pasted image 20240609143257.png|500]]
### Configuring CNI
쿠버네티스가 사용할 CNI 플러그인을 어디에 지정해야 할까?
-> *kubelet*
![[Pasted image 20240609142805.png|500]]
set to cni
컨테이너를 생성한 후 적절한 네트워크 플러그인을 호출해야하므로 컨테이너 생성을 담당하는 kubernetes의 kubelet에서 cni 플러그인을 구성한다.
각 노드에 있는 kubelet.service에서 cni 플러그인을 설정했음을 확인 할 수 있다.

![[Pasted image 20240609142814.png|500]]
cni bin, config directory
![[Pasted image 20240609142827.png|300]]
https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#support-hostport
- isGateway: 브리지 네트워크가 gateway 역할을 할 수 있도록 IP 주소를 할당받아야 하는지의 여부정의
- isMasq: IP 위장을 위한 NAT rule을 추가해야하는지 여부 정의
- ipam(ip address management): 파드에 할당할 서브넷 대역을 지정.host local 유형은 ip주소가 호스트에서 로컬로 관리되고 유지됨을 나타낸다. 유형을 DHCP로 설정하여 외부 DHCP서버를 구성할 수도 있다.

(ipam: ip 주소를 자동으로 부여하고 쉽게 찾을 수 있도록 지원하는 미들웨어 역할을 하는 솔루션)

### CNI Weave
이전 강의에서 수동을 설정한 네트워킹 솔루션에서는 어떤 네트워크가 어떤 호스트에 있는지 매핑하는 route table이 존재했다.  이는 소규모의 단순한 네트워크에서는 잘 작동하겠지만, 수백개 파드가 있는 대규모 환경에서는 실용적이지 못한다. 
route table은 너무 많은 항목을 지원하지 않을 수 있기 때문에 별도의 솔루션을 사용하는 것이 좋다.
앞으로 weave 라는 솔루션에 대해 알아봐보자.

Weave 솔루션은 어떻게 작동할까?
![[Pasted image 20240609173201.png|500]]
예를 들어 kubenetes cluster을 회사로 노드를 서로 다른 사무실로 생각해보자.
1번 사무실에 있는 누군가가 3번 사무실로 소포를 보내고자 한다했을때,
배송을 직접하는게 아닌 이를 가장 잘하는 운송회사에게 아웃소싱하는 것과 비슷하다.

운송회사와 계약하면, 가장 먼저하는 것은 그들의 에이전트를 각 사무실마다 배치하는 것이다.
이러한 에이전트는 사무실 간의 모든 배송작업을 관리한다. 그들은 또한 서로 계속 이야기하고 연결되어있다.
따라서 소포를 보내면 그 사무실의 운송업자가 소포를 가로채서 대상 사무실 이름을 본다. 
그는 다른 사무실의 에이전트들과 작은 내부 네트워크를 통해 그 사무실의 정확한 위치를 알고나서,
대상 목적지 주소가 설정된 자신의 *새패키지*에 해당 패키지를 넣은 다음 보낸다.
패키지가 해당 사무실의 에이전트에게 도착하면 그 에이전트는 패킷을 열고 원본 패킷을 꺼내어 올바른 부서로 전달한다.


![[Pasted image 20240609173211.png|500]]
위의 예시와 같이 weave CNI 플러그인을 클러스터에 배포하면,
각 노드에 에이전트 또는 서비스를 배포한다.
이들은 서로 통신하여 노드와 노드내의 네트워크 및 파드에 대한 정보를 교환한다.
그리고 weave는 노드와 namespace에 자체 브리지를 생성하며, 각 네트워크에 IP 주소를 할당한다.

*하나의 파드가 여러 브리지네트워크에 부착할 수 있기 때문에*
docker 에서 만든 docker bridge 뿐만아니라 weave bridge에도 파드를 부착할 수 있다.

패킷이 한파드에서 다른 노드의 다른 파드로 전송할때, weave는 패킷을 가로채고 해당 패킷을 새 패킷으로 *캡슐화*하고 별도의 네트워크를 통해  전송한다. 이를 전송받은 다른 weave 에이전트는 캡슐을 해제하고 패킷을 목적지 파드로 라우팅한다.

![[Pasted image 20240609173221.png|500]]
weave는 수동으로 클러스터의 각 노드에 서비스 또는 데몬으로 배포할 수 있으며,
kubernetes가 이미 설정되어있는 경우, 클러스터의 파드로 배포하는 것이 더 쉬운 방법이다.

![[Pasted image 20240609173229.png|500]]

#### test
![[Pasted image 20240609175920.png|500]]
![[Pasted image 20240609181056.png|500]]

weave install
https://kubernetes.io/docs/concepts/cluster-administration/addons/
https://github.com/rajch/weave#using-weave-on-kubernetes
![[Pasted image 20240609181828.png|500]]

### IPAM Weave
ip address management
CNI 
- CNI는 컨테이너에 IP를 할당하는 책임이 있다.
![[Pasted image 20240609182526.png|500]]
각 노드의 가상 브리지 네트워크에 IP 서브넷을 할당하는 방법?
중복된 IP가 할당되지 않았는지 어떻게 체크할까?
이전 강의에서 스크립트로 네트워크를 구성하는 과정에서 IP list 생성해서 관리할 수도 있을 것이다. 이 ip list 파일이 각 호스트에 배치되어 관리할 수 있을 것이다.

하지만 이렇게 직접 스크립트를 구성하는 것보다 CNI에는 이 작업을 아웃소싱 할 수 있는 plugin이 built in 되어있다.
- DHCP
- host local
두 플러그인중 어떤걸 호출할지는 선택하기 나름이다. 
또한 스크립트를 동적으로 만들어 이 이외의 다양한 종류의 플러그인을 지원 할 수도 있다. 

![[Pasted image 20240609182537.png|500]]
CNI configuration 파일에는 사용할 플러그인 유형, 서브넷 및 경로를 지정할 수 있는 IPAM이라는 섹션이 있는데
이때 어떤 플러그인을 사용할지 지정한다. 

#### Weaveworks가 IP 주소를 어떻게 관리방법
 - default로 Weave는 전체 네트워크에 대해 IP 범위 `10.32.0.0/12`를 할당한다. 
 - 네트워크 IP 범위는 `10.32.0.1 ~ 10.47.255.254` - 네트워크의 파드에 사용할 수 있는 IP는 약 백만 개. 
 - IP 주소를 동일하게 분할하여 각 노드에 한 부분을 할당하여 사용.
 - 이러한 범위는  Weave 플러그인을 클러스터에 배포하는 동안 이전의 추가 옵션으로 구성할 수 있다.

#### test
What is the Networking Solution used by this cluster? (네트워킹 솔루션이 뭔지?)
![[Pasted image 20240609183527.png|200]]

how many weave agents/peers
![[Pasted image 20240609183618.png|500]]
the name of bridge network interface
![[Pasted image 20240609183933.png|500]]

weave pod ip address 대역 
![[Pasted image 20240609184227.png|500]]
![[Pasted image 20240609184246.png|500]]

default gateway configured on the pods scheduled on node01
`kubectl run busybox --image=busybox --dry-run-client -o yaml -- sleep 1000 > busybox.yaml`
`vi busybox.yaml`
-> nodeName: node01 추가
k apply
`kubectl exec busybox -- ip route`
으로 route 조회

## Service Networking

파드간의 networking에 대해 살펴봤지만, 파드가 직접 통신하도록 구성하는 경우는 *거의 없다*
다른 파드에서 호스팅 되는 서비스에 엑세스하려면 항상 서비스를 사용해야한다.
#### Cluster IP
만약 블루 파드가 오렌지 파드에 접근하도록 하기 위해 오렌지 서비스를 만든다면,
오렌지 서비스는 IP 주소와 Name을 할당받고 블루파드는 오렌지서비스의 IP또는 Name을 통해 오렌지 파드에 엑세스 한다. 

만약, 다른 노드에 있는 파드에 엑세스 한다면?
*서비스가 생성되면 파드가 어떤 노드에 있는지 관계없이 클러스터의 모든 파드에서 서비스에 엑세스 할 수 있다.*
*파드는 노드에 호스팅되지만, 서비스는 클러스터 전체에서 호스팅된다*
-> 이 서비스는 특정 노드에 바인딩 되지 않고 클러스터 내에서는 엑세스 할 수 있다. 
-> 이를 *Cluster IP*라고 한다. 

![[Pasted image 20240609184946.png|500]]
퍼플파드가 외부에서 엑세스 할 수 있도록 한다면, 
*NodePort* 서비스를 생성 할 수 있다.
이 서비스 또한 IP 주소를 할당받고 Cluster IP와 마찬가지로 작동한다. 
다른 모든 파드에서는 IP를 사용하여 서비스에 엑세스 할 수 있다. 
각 노드에 배치한 포트를 통해 외부로부터 서비스에 엑세스가 가능하다.

**서비스는 어떻게 IP주소를 할당받고 클러스터의 모든 노드에서 해당 네트워크를 사용할 수 있는 것일까?**
![[Pasted image 20240609190351.png|500]]
#### pod
kubelet 은 kubeapi server를 통해 클러스터 변경사항을 모니터링 하며 새로운 파드가 생성요청이 올때마다 노드에 파들 생성한다. 그다음, CNI 플러그인을 호출하여 해당 파드에 대한 네트워킹을 구성한다.
#### service
kube-proxy는 kubeapiserver를 통해 클러스터 변화를 감지하고, 새로운 서비스가 생성될때마다 Kube-proxy가 실행된다.
*파드와 달리 서비스는 각 노드에 할당되지 않고 클러스터 차원의 개념이다.*
클러스터의 모든 노드에 존재한다. 
사실 파드의 경우 컨테이너가 있고 인터페이스와 IP가 할당된 네임스페이스가 있지만, 서비스는 그런것이 없고, 단지 가상의 물체일 뿐이다. 

**그럼 어떻게 IP 주소를 얻고 서비스를 통해 파드에 엑세스할까?**
![[Pasted image 20240609190412.png|500]]
서비스를 생성하면 *미리정의된 범위의 IP주소가 할당된다*
#### To check the Service Cluster IP Range
ClusterIP를 생성되면, IP 주소를 할당하는데 Cluster IP Range는 kube api 서버 옵션에 지정되며, default로 `10.0.0/24`로 설정된다.
이때 파드네트워킹의 경우 ` 10.244.0.0/16`로 설정되어있었는데 - 각 네트워크에 어떤 범위를 지정하든 *중복되지 않아야한다.*

해당 노드에서 실행중인 kube-proxy 컴포넌트는 해당 IP주소를 가져오고 클러스터의 각 노드에 rule을 생성한다.
해당 rule -> 서비스의 IP인 이 IP로 오는 트래픽은 파드IP로 가야한다. 는 rule 이다.
따라서 서비스가 생성되면 파드가 서비스의 IP에 도달하려고 할때마다 파드의 IP주소로 전달되게 되는 것이다.
*IP 뿐만 아니라 Port 조합도 가능하다*
*서비스가 생성 삭제될때마다 kube-proxy는 이러한 rule을 생성 삭제한다.*

이러한 rule들은 
- ipvs
- userspace
- iptables
를 사용하곤 하는데 대부분 *iptables*를 사용하곤 한다.
이외에 kubeproxy 를 구성할때 프록시 모드 옵션을 설정하여 정할 수 있는데,
default는 *iptables이다.*

#### To check the rules created by kube-proxy in the iptables
iptables를 조회해보면, rule들을 볼 수 있는데,
서비스 IP로 들어오는 트래픽은 해당 파드IP로 전달되는 rule을 확인 할 수 있다.
그리고 이 작업은 iptables 에 DNAT 를 추가하여 수행된다. 목적지가 파드IP로 변경되는 것이다. 
-> *ClusterIP*

마찬가지로 *NodePort* 유형의 서비스를 생성할 때, 
Kube-proxy는 모든 노드의 포트에서 들어오는 모든 트래픽을 해당 백엔드 파드로 전달하는 IP 테이블 rule을 생성한다.

![[Pasted image 20240609190445.png|500]]
로그에서도 해동 rule에 대해 확인 해 볼 수 있다.

#### test

What network range are the nodes in the cluster part of?
`ip add | grep eth0`

What is the range of IP addresses configured for PODs on this cluster? ( 클러스터의 POD에 대해 구성된 IP 주소 범위)
`kubectl logs weave-~~ -n kube-system`

What is the IP Range configured for the services within the cluster? (클러스터 내의 서비스에 대해 구성된 IP 범위)
`cat /etc/kubernetes/manifests/kube-apiserver.yaml`
