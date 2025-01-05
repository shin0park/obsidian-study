
## Admission Controller
전시간에 Authentication-KubeConfig, Authorization-RBAC를 배웠는데 
RBAC으로는 어느 리소스에 대해 사용자의 access를 정의하는것 이상은 안된다. 
그이상을 원한다면???
만약 pod를 생성할때 
pod의 configuration file을 검토하고 이미지 이름을 public docker hub registry의 이미지를 허용하고 싶지 않다면 어떻게 해야 할까?
내부 특정 레지스트리의 이미지만 허용시키고 싶은 것이다. 
```
containers:
  name: ubuntu
  image: ubuntu:latest
```
그럴려면 ubuntu:latest 처럼 이미지에 최신 태그를 사용하거나 하면 안될 것이다. 

또는 컨테이너가 root 사용자로서 실행중이라면 
```
securityContext:
  runAsUser: 0
  capabilities:
    add: ["MAC_ADMIN"]
```
해당 요청을 허용하고 싶지 않을땐 어떻게 해야할까
또는 특정 capabilities 기능만 허용하거나
```
metadata:
  name: web-pod
```
metadata엔 항상 label을 포함해야된다던지

*이럴때 admissiooon Controller*를 사용한다. 

	kubectl -> Authentication -> Authorization -> Admission Contrllers -> create Pod

admission controller는 클러스터 사용방식을 더 강화하기 위한 더 나은 보안 조치를 시행하도록 도와준다. 

### view enabled admissioin controllers
```
kube-apiserver -h | grep enable-admission-plugins
```

kube-apiserver 가 서비스를 업데이트하는 경우 
kube-apiserver.service 파일에 
`--enable-admission-plugins=~~`
으로 플러그인을 추가하면되며 

api서버가 kube ADM 기반 셋업에서 포드로 실행되는경우 
`/etc/kubernetes/manifests/kube-apiserver.yaml`
의 pod 정의 파일에 containers: -command: 에 커맨드로 
`--enable-admission-plugins=~~`
추가하면 된다. 

```
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep --enable-admission-plugins
```

## **실습**

### 특정 admission-plugins을 작동안시키고 싶을땐 
`vi /etc/kubernetes/manifests/kube-apiserver.yaml`
로
![[../../resources/Pasted image 20230617171708.png|500]]
`--disable-admission-plugins=`에 추가한다.

### admission-plugins 확인하는 또다른 방법 
`ps -ef | grep`
`ps -aux | grep`

![[../../resources/Pasted image 20230617171908.png]]
![[../../resources/Pasted image 20230617171900.png]]

### Which admission controller is not enabled by default?
```
kubectl exec -it kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep 'enable-admission-plugins'
```
이라 치면 default admission controller 볼 수 있다.

#### enable-admission-plugins 보는 또다른 방법 
`grep enable-admission-plugins /etc/kubernets/manifestes/kube-apiser.yaml`


## Validating&Mutating Admission Controller
간략하게 말하자면 
```
mutating과 validating으로 나뉜다.  
#### mutating
만약 그 레이블이 안들어가 있으면 설정하고 배포한다.
#### validating
레이블 없으면 그냥 오류를 내보낸다.
```
앞에서
admission controller 에 NamespaceExists 플러그인으로 ns가 있으면 pass하고 없으면 reject하는 걸 볼 수 있다. 이는 입구를 관리하는 장치로 *Validating Admission Controller*이다.

defaultStroageClass의 경우 PVC 생성요청이 왔다고 하자 그럼 인증인가 단계를 거쳐 admission 단계에 왔을때 PVC 요청임을 확인하고 StorageClass가 언급되어있는지 확인한다. 없다면 true로 request를 수정해서 요청에 기본저장소를 추가할 것 이다. 
so 생성후 
`k des pvc myclaim`
확인해보면 strorageClass: default
로 되어있을 것이다. 
이런유형의 admission controller는 
*Mutating Admission Controller*라 한다.
생성되기 전에 개체 자체를 변경하거나 특정부분을 변경할 수 있다.

즉 **Mutating Admission Controller 는 요청을 변경할 수 있고 Validating Admission Controller는 유효성을 검사해 허용 또는 거부한다.**
그리고 **둘다 할 수 있는 Admission Controller도 있다.**

보통 Mutating이 먼저 실행되 변형이 먼저되고 그다음 Validating이 실행되어 유효성을 확인한다. 

default로 되어있는 Admission controller 만이 아닌
외부에서 우리만의 자체 Mutating Validating admission controller를 원한다면?

이는 바로 **Mutating Admission Webhook Validating Admission Webhook**을 사용하면 된다. 

1. 고유로직이 있는 admission webhook server를 배포한다. - *Deploy Webhook Server*
   어떤 플랫폼에서든 구축할 수 있는 API서버가 될 수 있다.
2. webhook configure 생성해 kubernetes에 webhook을 구성한다. - *Configuring Admission Webhook*
   즉 클러스터를 구성한다. 
   deploy된 webhook-deployment가 있다면 그걸 둘러싼 kubernetes cluster가 있고 이와 연결하기 위한 service인 webhook-service를 만들어야될 것이다. 서비스에 connect해 요청의 유효성을 체크하고 변형시키기 위해 있는 것이다.


![[../../resources/Pasted image 20230617185804.png|500]]
clientConfig : 배포하는 위치 
도메인이면 도메인 작성 
다른 서비스에 배포한다면 위에처럼 서비스이름과 ns를 넣으면된다.

rules
api 서버를 호출할때 지정
위에선 pod가 생성될때만 이 구성을 호출한다고 보면된다. 
pod가 생성될때마다 서비스에 호출이 발생한다. 

