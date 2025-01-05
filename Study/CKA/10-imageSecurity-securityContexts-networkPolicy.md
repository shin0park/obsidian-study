## Image Security

pod 정의파일에서
spec.containers 하위에 image 속성을 입력한다.
ex) 
```
image: nginx
```
여기서 Image name은 docker의 이미지 명명 규칙을 따른다. 
여기서 nginx는 image/repository (이미지 혹은 저장소) 이름이다.

![[Pasted image 20240519221950.png|500]]
*User/Account* 를 지정하지 않으면 library라고 인식한다.
*library*: docker 공식 이미지가 저장되는 default 계정 이름이다.  
*Registry*: 어디서 이미지를 가져올 것인지 지정. 레지스트리는 all of image 를 저장하는 곳이다. 이미지를 생성하거나 업데이트할때, 레지스트리에 푸시하면 누군가 pull 하여 사용하는 것. 
기본값 레지스트리는 Docker Hub -> docker.io
google -> gcr.io

이들은 누구나 공개적으로 접속하여 사용할 수 있는 이미지들이다. 
일반인에게는 엑세스되면 안되는 경우, 내부에서 사용하는 *internal private Registry*를 사용할 수 있다. 
AWS, Azure, GCP 같은 많은 클라우드 서비스 제공자는 기본값으로 private registry를 제공한다.

![[Pasted image 20240519221957.png|500]]
Docker hub, Google 레지스트리 또는 내부 사설 레지스트리에서 *internal private repository*를 만들 수 있다. 

![[Pasted image 20240519222009.png|500]]
#### how to implement authentication?
어떻게 쿠버네티스가 이미지에 엑세스하기 위한 credential을 얻을 수 있을까?
in kubernetes, 워커노드에서 docker 런타임에 이미지를 pull 받는다.
그럼 어떻게 credential을 워커노드의 docker 런타임에 전달할 수 있을끼?
-> `kubectl create secret docker-registry (name)`
docker-registry : built-in secret type

그다음, 옵션으로
--docker-server= 레지스트리 서버이름
레지스트리에 엑세스할 username, pwd, email을 입력한다.  

`imagePullSecrets` 옵션에 앞서 생성한 secret 이름을 지정해주면된다.
-> pod가 생성되면 워커노드의 kubelet은 secret 로부터 credential을 가져와 image pull 하는데에 사용한다.


#### test
```
k create secret docker-registry private-reg-cred --docker-server=myprivateregistry.com:5000 --docker-username=dock_user --docker-password=dock_password --docker-email=dock_user@myprivateregistry.com
secret/private-reg-cred created
```

### Prerequisite - Security in docker

docker가 설치된 host가 있다고하자.
![[Pasted image 20240519234525.png|200]]
`docker run ubuntu sleep 3600`
로 sleep 프로세스를 동작하는 ubuntu docker container를 생성

vm(virtual machine)과 달리 container는 host와 완전히 격리되어있지 않다. 
*컨테이너와 호스트가 같은 커널을 공유한다.*
컨테이너는 linux의 namepace 를 통해 격리된다.  

host와 컨테이너는 각각 namepace가 존재하고,
컨테이너에 의해 실행되는 모든 프로세스는 사실 호스트 자체에서 실행되지만 고유의 네임스페이스에서 실행되기 때문에 마지 완전히 분리된것 처럼 느껴진다.

#### Process isolation
docker 프로세스에서는 고유의 namepace만 볼 수 있으면 바깥의 다른 namepace의 리소스는 볼 수 없다.
`ps aux`
앞서 생성한 docker 컨테이너의 프로세스 리스트 나열
pid 1인 sleep process 를 볼 수 있다.

만약 호스트에서 
`ps aux`
로 프로세스 리스트를 조회하면
![[Pasted image 20240519234618.png]]
하위 자식 namepace 의 프로세스도 모두 조회할 수 있음을 볼 수 있고, 위의 sleep process도 포함됨을 볼 수 있다. 
이는 프로세스가 다른 namepace의 다른 process id를 가질 수 있기 때문이다. 이것이 docker가 시스템 내에서 컨테이너를 격리하는 방식이다.
-> *process isolation*

#### User in Context Security 
docker host는 
Root user와 nonRoot user를 가진다.
`default : Root User`

`ps aux`
실행해보면 기본값으로 USER가 root 로 되어있음을 볼 수 있다.

*User 변경*
1. `docker run --user=1000 ubuntu sleep 3600`

2. in dockerfile,
	```
	FROM ubuntu
	USER 1000
	```
	
	`docker build -t my-ubuntu-image .`
	`docker run my-ubuntu-image sleep 3600`

#### Root User
Root user는 시스템의 가장 강력한 사용자이다. - 시스템에 제한없이 접근할 수 있다. 

*container 안의 root user와 host 안의 root user는 다르다.*
host의 root user가 할 수 있는 것을 container root user도 모두 할 수 있다면, 보안상 안될 것이다.
docker는 이를 구현하기 위해 linux의 기능을 사용한다. -> linux capability

#### Linux Capability
![[Pasted image 20240519234714.png|400]]
루트 사용자는 시스템에서 가장 강력한 사용자이다. 루트 사용자는 뭐든 할 수 있고 루트 사용자에 의한 프로세스도 마찬가지로 시스템의 제한 없이 접근할 수 있다. 
`user/include/linux/capability.h` 
에서 루트 사용자가 할 수 있는 수많은 리눅스의 cap을 확인 할 수 있다. 
or
![[Pasted image 20240519233913.png]]
https://limitrequestbody.com/linux-capabilities-%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%BA%90%ED%8D%BC%EB%B9%8C%EB%A6%AC%ED%8B%B0-297ae87cb3c

이런 cap들을 통해 사용자가 사용할 수 있는 기능을 제어, 제한 할 수 있다.

**기본값으로 docker는 컨테이너의 기능을 제한한다.**
컨테이너에서 실행되는 프로세스는 호스트를 재부팅하거나 다른 컨테이너에 영향이 가는 작업을 하는 특권을 가질 수 없다.
만약 추가적인 권한을 갖고 싶다면 docker --cap-add 옵션으로 지정해야한다. cap drop 옵션도 존재한다.
`docker run --cap-add MAC_ADMIN ubuntu` 
`docker run --cap-drop KILL ubuntu` 
모든 권한이 활성화된 컨테이너를 실행하고 싶다면 `--previleged` 옵션을 사용하면 된다 


### Security Contexts
지난 강의에서 봤듯이 docker 컨테이너를 실행할 때 보안 표준 집합을 정의할 수 있었다.

![[Pasted image 20240519235022.png]]
쿠버네티스에서도 동일하다.
쿠버네티스에서는 컨테이너를 파드에 넣어 보관하기 때문에, 컨테이너 수준이나 파드 수준에서 보안 설정을 선택할 수 있다.

*파드 수준으로 설정하면 파드 내 모든 컨테이너에 설정값이 전달된다.*

파드 정의 파일에서 **securityContext** 영역을 통해 파드 수준의 보안 설정을 할 수 있다.  
**runAsUser** 필드으로 파드 사용자 ID를 설정한다. 

![[Pasted image 20240519235033.png]]
**컨테이너 수준에서 보안을 설정하려면 containers 영역에 securityContext 영역을 추가하면 된다.**  
그리고 **capabilities 영역**에 추가할 권한 목록을 명시하면 된다. 
*capabilities 영역은 컨테이너 수준에서만 지정 가능하다*

#### test
What is the user used to execute the sleep process within the `ubuntu-sleeper` pod?
`kubectl exec ubuntu-sleeper -- whoami`

##### pod level
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
```

##### container level
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper
  namespace: default
spec:
  containers:
  - command:
    - sleep
    - "4800"
    image: ubuntu
    name: ubuntu-sleeper
    securityContext:
      capabilities:
        add: ["SYS_TIME", "NET_ADMIN"]
```



### Network Policy
#### Traffic
Basic flow and rule
![[Pasted image 20240520000058.png|500]]
ingress : 입구, 들어오는 
egress: 퇴장, 나가는

web 80포트로 db를 거쳐 통신하는데 필요한 ingress, egress를 나열하면 아래와 같다.
![[Pasted image 20240520000108.png|500]]
이것이 트래픽 흐름과 rule의 기본이다.

#### Network Policy

![[Pasted image 20240520000557.png|500]]
쿠버네티스 환경에서는 파드와 노드 그리고 서비스가 존재하며 각각 IP 주소가 존재한다.
##### 쿠버네티스 네트워킹의 전제조건  
1.  파드가 서로 내부적으로 통신할 수 있어야 한다. (서로 다른 노드의 파드끼리도)

모든 파드가 가상 사설 네트워크에 있고 IP나 파드 이름 또는 서비스 이름을 통해 
**쿠버네티스 클러스터 내 노드를 가로 지르며 서로서로 통신 할 수 있다.**
따라서 쿠버네티스는 기본값으로
*All Allow*
를 제공한다.

![[Pasted image 20240520000606.png|500]]
따라서 앞선 환경에서 각각 pod로 배포를 했다면 위의 이미지와 같이 서로 통신할 수 있을 것이다. 
이때 만약 web프론트와 db가 직접적으로 통신하는 것을 막기위해서는 어떻게 해야할까?
이때 *Network Policy*를 사용할 수 있다. 
이는 쿠버네티스의 객체로 정의하여 사용할 수 있고,
*하나이상의 파드에 네트워크 정책을 연결한다.*

![[Pasted image 20240520000613.png|500]]
방법은 위와 같이 label을 통해 적용할 파드를 지정하면 된다.

![[Pasted image 20240520000618.png|500]]
policyTypes에 트래픽이 Ingress인지 Egress인지를 명시하고 규칙을 지정한다.

Network Policy의 정의 파일은 다음과 같다.  
이 Network Policy는 진입 트래픽만 격리되고 퇴장 트래픽은 영향이 없다.

![[Pasted image 20240520000624.png|500]]

![[Pasted image 20240520001306.png|400]]
**Network Policy는 Network Solution에 의해 실행된다. 
모든 Network Solution이 Network Policy을 지원하는 것은 아니다.**  
네트워크 정책을 지원하는지 확인하기 위해 공식 문서를 항상 참조하길 권한다.
네트워크 정책을 지원하지 않는 솔루션으로 구성된 클러스터에서도 정책을 만들 순 있지만 네트워크 솔류션이 지원하지 않는다는 오류메시지는 나오지 않을 것이다.

### Developing network policies

백엔드 API 파드를 제외한 다른 리소스로부터 db 파드로의 트래픽을 허용하지 않고자 한다.

**만약 들어오는 트래픽을 허용하면 그 트래픽에 대한 응답이나 회신이 자동으로 허용된다. 
-> 응답, 회신에 대한 것은 따로 규정하지 않아도 된다.**

![[Pasted image 20240520001637.png]]
이때 만약 다른 네임스페이스에 있는 api pod도 db와 통신하고자 한다면 어떻게 해야할까?

![[Pasted image 20240520001645.png]]
**namespaceSelector 영역에 label을 통해 namespace를 지정하면된다.**
이때 namespace 에도 라벨을 설정해야한다!.

podSelector 없이 namespaceSelector만 지정할 수도 있다.  
그럴 경우, 지정한 네임스페이스 내의 모든 파드는 DB 파드에 도달할 수 있게 된다.

![[Pasted image 20240520001657.png]]
만약 외부 백업 서버가 db 파드에 접속하고 싶은 경우는 어떻게 해야할까?
클러스터 외부에서 데이터베이스 파드에 접속할 수 있도록 네트워크 정책을 구성할 수 있다.  
ipBlock 영역을 통해 IP 주소를 지정하면 해당 IP의 서버가 DB 파드에 접근할 수 있다.

**즉, 특정한 Pod, Namespce, IP 주소를 지정하기 위해 podSelector, namespaceSelector, ipBlock 영역을 사용하면 된다.**  
이는 개별적인 규칙으로 통과될 수도 있고, 하나의 규칙의 일부로 함께 통과될 수도 있다.

`-`로 podSelector와 namespaceSelector가 함께 적용되있다면,
이로써 첫 번째 규칙이 만들어지고 ipBlock을 통해 두번째 규칙이 만들어진다.
-> 즉, `-`로  정의한 rule은 *AND*로 동작한다. 

만약 namespaceSelector 앞에 **대시를 추가해 규칙을 분리한다면.**  위 이미지와 같이
첫 번째 규칙은 podSelector에 의해 api-pod와 일치하는 모든 파드가 허용
두 번째 규칙은 namespaceSelector에 의해 prod 네임스페이스 내의 어떤 파드에서든 트래픽이 허용
세 번째 규칙은 특정 IP를 가진 개체를 허용하는 규칙이 적용되는 것이다.

![[Pasted image 20240520001709.png]]
반대로 DB 파드에서 백업 서버로 푸쉬해야 한다면 db 파드에서 외부 백업 서버로 네트워크를 허용해야 한다.  
policyTypes에 Egress를 추가하고 
db 파드에서 외부 백업 서버로 통신을 허용하는 egress 규칙을 추가해야 한다.


Tip: Kubectx and Kubens – Command line Utilities

#### test
How many network policies do you see in the environment? ( Network Policy 조회 )
`kubectl get networkpolicies`
`kubectl get netpol`

![[Pasted image 20240520004339.png|400]]
