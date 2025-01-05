## KubeConfig

![[Pasted image 20240512214454.png|500]]
앞서 Certificate에서 다뤘듯 kubeapiserver에 api를 요청하는경우, kubectl 명령어를 입력하는 경우 등 해당요청에 인증과정이 필요한데 그럴때마다 option으로 crt, key, ca.crt 를 모두 입력해주는 것은 번거로운 작업이다.
따라서 *kubeconfig* file에 이를 설정하여 손쉽게 사용할 수 있다.
kube controller는 기본적으로 `$HOME/.kube/config`에서 config 파일을 찾아 적용한다. 
kubectl 명령의 파일경로를 명시적으로 지정할 필요가 없다.
지금까지 config 파일 위치를 따로 지정하지 않아도 동작했던 이유이다.

![[Pasted image 20240512214841.png|500]]
#### kubeConfig file 구성
- Clusters
  쿠버네티스의 클러스터의 정보를 입력하는 구간으로, *멀티클러스터 환경*인 경우 클러스터 정보가 여러개 일것이다. 
  -> clusters
- Contexts
  clusters와 users의 조합.
  어떤 사용자 계정이 어떤 클러스터에 엑세스 하는지 정의한다.
- Users
  클러스터에 엑세스 권한이 있는 *사용자 계정* 정보이다

기존에 옵션으로 줬던 값이 config file에는 어느 구간에 들어갈까?
Clusters
- --server my-kube-playground:6443
- certificate-authority ca.crt
Users
- --client-key admin.key
-  --client-certificate admin.crt


![[]]
![[Pasted image 20240512215314.png|500]]
이와 같이 kubeconfig file을 설정하면 추가적인 object 생성할 필요 없이
kubectl 명령에 의해 자동으로 읽히고 필요한 값이 사용된다.

그럼 kubectl 많은 contexts들 중에 어느 context로 인증을 해야할지 어떻게 알 수 있을까?
-> *current-context*

![[Pasted image 20240512214940.png|600]]
따로 어느 kubeconfig 파일을 사용할지 지정하지 않으면 default file location 에 있는 kubeconfig 파일을 사용한다.
`-kubeconfig` 옵션을 통해 config를 지정하여 view도 가능하다.

![[Pasted image 20240512214932.png|600]]
how to change `current-context`?
`kubectl config use-context (contextname)`

![[Pasted image 20240512220226.png|500]]
특정 namespace를 위해 context를 구성할 수도 있다.
contexts에 namespace를 지정하면, 해당 context로 전환하면 특정 namespace로 들어가게 된다.

![[Pasted image 20240512220248.png|500]]
certificate 파일을 지정할때는 위 이미지와 같이 경로로 지정하면 적용된다.
단, 경로로 지정하지 않고,
`certificate-authority-data`
field 에 직접 crt data(base64로 인코딩한)를 넣을 수도 있다.

## API Groups
What is kubernetes API?
 쿠버네티스 관련 작업을 하기 위해서는 kube-apiserver와 통신이 필요하다.
 
![[Pasted image 20240512221700.png|600]]
마스터 노드의 apiserver 엑세스하여 version을 확인 할 수있다.
`api/v1/pods`를 통해서 접속가능한 pod 를 조회할 수도 있다.

![[Pasted image 20240512221730.png|500]]
쿠버네티스 API는 목적에 따라 그룹으로 그룹화가 된다.
version: 클러스터 버전 확인
metrics, healthz: 쿠버네티스 상태 모니터링
log: logging application과의 integration 

![[Pasted image 20240512221735.png|300]]
API는 두가지 그룹으로 나뉜다. core, named

![[Pasted image 20240512221712.png|500]]
core 그룹에는 핵심 기능들이 들어있다.

![[Pasted image 20240512221720.png|500]]
좀더 organized 되어있다. 새로운 기능들은 apis 그룹을 통해 사용 가능해질 것이다.
*API Groups -> Resources -> Verbs*

![[Pasted image 20240512222550.png|500]]
다음과 같이 curl을 통해 API에 엑세스하는 경우, 인증메커니즘을 지정해주지 않으면 다음과 같이 실패가 뜬다.

![[Pasted image 20240512222828.png|500]]
kubectl proxy로 클라이언트를 실행할 경우,
`kubectl proxy` command는 localhost:8001 에서 프록시 서비스를 실행하고, 클러스터 엑세스를 위해 kubeconfig file의 cerificate을 사용하여 접속한다.
*kube proxy != kubectl proxy*
kubectl proxy는 ACTP 프록시 서비스로 kubectl 유틸리티가 kube-apiserver에 엑세스 하기 위해 만든 것.

## Authorization
인증에 대해 이야기 했다면,
접속하면 어떤 것을 할 수 있는지를 정의하는 
인가에 대해 알아보자.

관리자, 개발자, bots 등 어떤 사람이 엑세스하는가에 따라 다른 권한들을 가져야한다.

![[Pasted image 20240512223349.png|500]]

![[Pasted image 20240512223358.png|500]]
[Access in Cluster]
클러스터 내 관리 프로세스를 위해 클러스터 내 노드에서도 kubelet이 kube API 와 통신한다.
api 서버에 엑세스하여 services, pods 등의 리소스를 read하고 write를 한다.
또한 kube API 서버에 노드에 대한 정보를 보고한다. 상태정보
이런 요청은 `Node Authorizer`가 처리한다.
해당 노드의 certficate를 통해 요청을 하면, Node Authorizer가 승인 하고 이런 특권을 부여한다.


![[Pasted image 20240512224352.png|500]]
[API의 외부 Access]
사용자, 사용자 그룹과 permission으로 연결해야한다. (누가 어떤 권한을 갖는지)
policy 정책들을 만들어야하며, 이를 API 서버에 넘기는데
새로운 정책을 추가 수정할때마다, Kube API 서버를 restart 해야한다.
ABAC은 관리하기 어렵다.

![[Pasted image 20240512223418.png|500]]
더 쉽게 관리 할 수 있다.
 사용자와 권한을 연결하는 대신 
 Role을 정의한다.
 그리고 그 Role에 User를 연결한다.
 앞으로 사용자의 엑세스에 변화가 필요할때 마다 Role을 수정하기만 하면 된다.
 그럼 그 Role과 연결된 모든 Users에게 적용된다.
 -> *가장 표준적인 방법*
 
![[Pasted image 20240512223426.png|500]]
Authorization 메카니즘을 outsource에 위탁하는 경우
외부에서 권한을 관리.
open policy agent가 사용자의 허용여부를 결정

![[Pasted image 20240512223440.png|500]]
![[Pasted image 20240512225136.png|500]]
다음과 같이 --authorization-mode 에 적힌 순서대로 인가를 하는데,
만약 사용자가 권한을 받는 경우 Node는 노드에 대한 권한만 부여하기 때문에 다음단계인 RBAC으로 넘어가 권한을 부여하게 된다.
따라서 각 모듈이 요청이 거부할때마다, 체인으로 다음 모듈로 넘어가는 구조이다.
### RBAC

![[Pasted image 20240512225348.png|500]]
role을 만든다음, RoleBinding 리소스를 통해 User와 연결해준다.
subjects: User detail 입력
roleRef: 바인딩할 Role 입력

*Role의 rules 에서 resourceNames 옵션으로 리소스를 지정하여 리소스 제한할 수도 있다.*

![[Pasted image 20240512225411.png|500]]

![[Pasted image 20240512225422.png|500]]
권한 확인
`-as`
`-namespace`
#### test
`k create role developer --verb=list,create,delete --resource=pod`
`k create rolebinding dev-user-binding --role=developer --user=dev-user`
`k get pod dark-blue-app --as dev-user -n blue`

### Cluster Roles

![[Pasted image 20240512231313.png|500]]
role, rolebinding 은 namespaced 이다. namespace를 지정하지않으면 default namespace에서 생성된다.
하지만,
cluster Role의 경우 범위가 namespacer가 아닌 
*클러스터 범위 리소스*이다.

![[Pasted image 20240512231427.png|500]]

#### test
`k get clusterroles --no-headers | wc -l`
`k create clusterrole michelle-role --verb=get,list,watch --resource=nodes`
`k create clusterrolebinding michelle-rolebinding --clusterrole=michelle-role --user=michelle`
## Service Account
쿠버네티스 계정에는 두가지 유형이 있다.
- User: use by human
- Service : use by machine
  ex) 프로메테우스, datadog 같은 모니터링 서비스, jenkins
  
에를들어 모니터링 시스템이 있다고 했을때, 해당 응용프로그램이 쿠버네티스 API를 쿼리하려면 *인증*이 되어야한다.
이럴때 service account를 사용한다. 

`k create serviceaccount dashboard-sa`
`k get serviceaccount`
`k describe serviceaccount dashboard-sa`
sa가 생성되면 자동으로 토큰이 생성되는데, (describe에서 확인 할 수 있다.)
토큰이 자동 생성되었을때, secret object을 해당 토큰을 저장해서 생성한다.
secret object는 sa에 linked 된다.
-> `k describe secret dashboard-sa-token-kddm`
으로 secret을 확인 할 수 있다.
해당 토큰은 쿠버네티스API에 REST call 하여 인증하는데 사용된다.

#### SA RBAC
service account를 생성하여 RBAC을 이용해 올바른 권한을 부여할 수도 있다.
#### Mount
 예를 들어 프로메테우스 처럼, 쿠버네티스 환경에 호스트된 응용프로그램에 대한 Service Account의 경우
 수동으로 토큰을 제공할 필요 없이,
 해당 응용 프로그램이 있는 Pod에 토큰을 자동으로 Mount시킴으로써 토큰에 쉽게 접근하여 사용할 수 있게 구성되어있다. 

#### default SA
 - 쿠버네티스 모든 네임스페이스에는 
   *default로 명명된 Service Account가 자동 생성된다.*
   pod가 생성될때마다 default sa에 대한 토큰이 해당 pod에 자동 마운트된다.
	 `k describe pod my-dashboard` 로 살펴보면
	 따로 볼륨마운트를 설정한 적이 없지만,
	 volumes:, Mounts: 에 default-token 이 마운트 되어있음을 확인 할 수 있다. 
	  *default sa*는 기본 쿠버네티스 API 쿼리하는 권한만 갖고있다.
- 만약 pod 생성시 default sa가 아닌 다른 sa를 지정하고 싶다면,
  `spec.serviceAccountName` 속성을 지정하면 된다.
- 만약 default sa를 pod에 자동으로 마운트시키지 않고 싶다면,
  `automountServiceAccountToken: false` 설정을 하면된다.

### 1.22/1.24 Update

이전 버전에는 토큰에 유효기간이 없었다.
sa가 존재하는한 token은 유효

1.22 
Bound Service Account Token
하지만, 1.22 버전 이후부터는 *TokenRequestAPI*를 도입.
*TokenRequestAPI* 를 통해 일정 시간 제한이 있는 토큰 생성되며, pod 가 생성될때 service account admission controller가 해당 토큰을 볼륨마운트 설정한다.

![[Pasted image 20240513000103.png|500]]
https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/

1.24 이후
*reduction of secret-based service account token*
service account를 생성할때 원래 자동으로 secret object가 생성되고 해당 객체에 토큰이 저장되었었는데,
이젠, sa 생성시 secret 객체가 자동생성되지 않으며, 직접 생성해줘야만 한다.

즉, 1.24 버전 이후의 k8s를 사용한 클러스터로 작업중이라면   
ServiceAccount와 함께 수동으로 Token을 위한 Secret을 생성해야한다.
```
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: default
  name: drone-bot
```

1.24 버전까지는 위와 같이 ServiceAccount를 생성하면   
`drone-bot-token-j13b7`와 같은 스타일의 토큰이 자동으로 생성되었으나

1.24 이후 버전부터는 보안 강화를 위해 토큰을 자동으로 생성하지 않는다.
따라서 아래와 같이 별도로 Secret 오브젝트를 생성해야한다.

```
apiVersion: v1
kind: Secret
metadata:
  name: drone-bot-secret
  namespace: default
  annotations:
    kubernetes.io/service-account.name: drone-bot
type: kubernetes.io/service-account-token
```
출처: [https://ondemand.tistory.com/382](https://ondemand.tistory.com/382) [Cloud Computing On Demand:티스토리]
 
  

