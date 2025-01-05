
# Deployment

![[Pasted image 20240317192647.png]]

deployment를 통해 인스턴스들을 손쉽게 업그레이드 및 배포를 할 수 있다.
하나의 depolyment를 생성하면 하나의 replica set이 생성되고 replica set에서 설정한 개수만큼 pod가 생성되는 구조이다.

![[Pasted image 20240317193016.png]]

### test
- `kubectl describe po` 로 어떤 image를 사용했는지 봐도 되지만,
	`-o wide` 옵션으로도 확인 가능.
	![[Pasted image 20240317193827.png]]
- kind의 첫번째 글자는 대문자
- yaml 을 정의하지 않아도 cli로도 deployment 생성가능
	![[Pasted image 20240317194129.png]]
- po = pod
  deploy = deployment
  rs=replicaset
  svc=service
  ns=namespaces
  -A = --all-namespaces 
# Service

애플리케이션 안팎이나 다양한 구성요소 간의 통신을 가능하게 한다.
![[Pasted image 20240317233811.png|500]]
예를 들어 프론트앤드 파드, 백엔드 파드, 데이터베이스 파드와의 통신을 돕고 최종적으로 end user와 통신할 수 있게 한다.

### 외부와 통신을 어떻게 해야할까?
![[Pasted image 20240317234047.png|400]]
직접 ssh로 노드내부에 들어가 파드의 IP를 curl 하면 통신할 수 있는 것을 확인 할 수 있으나,
실제 외부에서 바로 접속할 수 있는 방법은 아니다.

## Service Types
1. NodePort
   노드의 port를 서비스를 통해 파드에 엑세스하고 이 노드포트를 통해 외부와 통신할 수 있게 한다.
2. ClusterIP
   클러스터 안에서 가상 IP를 만들어 서비스간 통신을 가능하게 한다.
3. LoadBalancer
## nodeport
 ![[Pasted image 20240317234221.png|400]]
서비스중 nodeport를 사용하면 위의 그림과 같이 ssh 접속없이 사용자가 외부에서 직접 통신할 수 있게 된다.
![[Pasted image 20240317235402.png]]
포트가 다음과 같이 세개가 있다.
웹서버가 실행중인 포드의 port는 80. 이것을 **target port**라고 한다.
서비스의 port는 80으로 이는 간단하게 port라고 한다.
그리고 노드 자체에 포트가 있는데 웹서버 자체에 액세스하는데 사용하는 포트이다. 이를 **nodeport**라고 한다.
노드포트는 **30000~32767** 범위에 존재한다.

**yaml**
yaml의 spec.ports 부분의 필수필드는 port 뿐이다. 
targetport를 명시안하면 port의 포트가 targetport와 일치하다고 인식하며,
nodeport를 명시하지 않는 경우, 유효한 범위중 자동으로 지정된다.
또한, ports에 list 형태인 걸로 보아 매핑할 수 있는 port는 여러개이다.

#### 어느 파드와 노드포트를 매핑할 것인가?
![[Pasted image 20240317235909.png]]
selector를 사용하여 연결하고자 하는 파드의 라벨을 지정한다.

![[Pasted image 20240317235952.png]]

![[Pasted image 20240318000106.png|400]]
다음 이미지와 같은 selector의 라벨을 가진 pod가 여러개일 경우 추가적인 구성 및 설정 없이도,
서비스는 자동으로 세개의 파드 모두 연결한다.
서비스는 내장된 로드밸런서를 통해 포드로 부하를 분산한다. (random 알고리즘 사용)

![[Pasted image 20240318000614.png|400]]
클러스터 내, 여러 노드에 파드가 분산되어있고 같은 노드포드를 사용하는 경우에도 추가적인 구성 및 설정 없이 
자동으로 클러스터 내 모든 노드에 걸쳐 서비스를 생성하고, 타깃 포트들을 같은 nodeport에 매핑한다.

**즉, 하나의 파드 or 하나의 노드의 여러개의 파드 or 클러스터 내 여러 노드든 Nodeport는 추가적인 구성없이 동일하게 생성된다**

## ClusterIP

pod들은 각각 IP를 갖고 있고 이는 static 하지 않다. pod는 언제든 죽고 새로 생성될 수 있으므로 IP가 수시로 바뀐다.
따라서 앱간의 통신할때 IP에 의존할 수 없다.
![[Pasted image 20240318001110.png|400]]
서비스를 통해 pod를 하나로 묶고 하나의 인터페이스를 통해 single 파드에 통신할 수 있도록 할 수 있다.
묶인 파드들중 요청은 무작위로 하나의 파드로 전달된다. 
위 이미지 처럼 프론트 백엔드 redis 등 용도에 따라 하나로 묶어서 통신할 수 있고,
각 계층은 통신에 영향을 주지않고 필요한대로 확장 또는 이동할 수 있다.

![[Pasted image 20240318001535.png|600]]

## LoadBalancer

![[Pasted image 20240318001756.png|600]]
노드포트를 통해 외부와 통신한다면 위의 이미지처럼 각각 노드마다 다른 ip들과 노드포트를 사용자가 직접 지정해줘야한다.

결국 사용자에게 단일 url을 제공해야될텐데,
이를 위해 따로 VM을 띄워 nginx를 설치하고 로드밸런서 구성해서 트래픽 부하를 설정하고 관리하는건 번거로운 일이다.
But, 클라우드 플랫폼을 사용한다면, 쿠버네티스는 해당 클라우드의 native loadbalancer통합을 지원한다. 
![[Pasted image 20240318002130.png|600]]
**지원되는 클라우드 플랫폼에 한하여 작동.

## Namespace
![[Pasted image 20240318003450.png|500]]
Isolation 을 위한 분리된 공간
dev와 prod 환경이 분리하여 구성할 수 있으며, 네임스페이스는 각각의 고유한 정책들을 가질 수 있다. 
![[Pasted image 20240318003621.png|400]]
다음과 같이 각각 리소스마다 할당량을 네임스페이스 별로 지정하여 한도이상을 사용하지 않도록 구성할 수도 있다.
![[Pasted image 20240318003801.png|500]]
현재 위치한 네임스페이스에서는 리소스 명만 명시하면되지만 다른 네임스페이스에 있는 리소스에 접속할때는
리소스명에 namespace와 svc.cluster.local 을 명시해줘야한다.
이것이 가능한 이유는 서비스가 생성될때 DNS가 자동이로 이를 추가하기 때문이다
![[Pasted image 20240318004019.png|500]]

how to use
`--namespace=(namespace name)`
option

how to create
![[Pasted image 20240318004129.png|400]]

![[Pasted image 20240318004231.png|500]]
config set-context 명령어를 통해 네임스페이스 옵션없이 특정 네임스페이스를 유지시킬 수 있다.

`--all-namespaces` : 모든 네임스페이스에서 보기

![[Pasted image 20240318004423.png|500]]


## Imperative vs Declarative

![[Pasted image 20240318005119.png]]
imperative: cli (step by step)
declarative: yaml

![[Pasted image 20240318005219.png|700]]

## Certification Tips - Imperative Commands with Kubectl

`--dry-run=client` : This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

`-o yaml`: This will output the resource definition in YAML format on screen.
#### POD
**Create an NGINX Pod**
`kubectl run nginx --image=nginx`

**Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)**
`kubectl run nginx --image=nginx --dry-run=client -o yaml`

#### Deployment
**Create a deployment**
`kubectl create deployment --image=nginx nginx`

**Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)**
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

**Generate Deployment with 4 Replicas**
`kubectl create deployment nginx --image=nginx --replicas=4`
`kubectl scale deployment nginx --replicas=4`
`kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml`
`kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`

#### Service
**Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**
`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml` 
(This will not use the pods labels as selectors, instead it will assume selectors as **app=redis.** 
So it does not work very well if your pod has a different label set.)

**Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:**
`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`
(This will automatically use the pod's labels as selectors)
Or
`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`
(This will not use the pods labels as selectors)

--> recommend going with the `kubectl expose` command.
If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.

### test
- `k run` 명령어로 pod 생성시 --port 옵션을 지정하지 않으면 container port는 None으로 파드생성.
