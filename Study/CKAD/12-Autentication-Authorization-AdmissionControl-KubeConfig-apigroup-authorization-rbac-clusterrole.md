
## Autentication-Authorization-AdmissionControl
인증-인가-허가

### 인증과 인가 방법 이해
사용자, 그룹, 서비스 어카운트의 요청이 쿠버네티스에서 처리되기 위해서는 인증 ⇒ 인가 ⇒ 진입 제어의 세 단계를 통과해야 합니다.  
예를 들어 여러분들이 ‘k get po -A’라는 명령으로 모든 네임스페이스의 파드 리스트를 확인하는 경우를 생각해 봅시다.  
먼저 명령을 내린 머신에 있는 kubernetes config파일(기본적으로 $HOME/.kube/config파일)에서 현재 사용자가 누구인지를 찾습니다.  
‘k config view’라는 명령으로 보면 현재 컨텍스트의 현재 유저를 볼 수 있습니다.  
그 유저를 기본인증, X.509 인증서 기반 인증, OAuth 연동 인증, WebHook 연동 인증, 프락시 서버 연동 인증 중 하나의 방법으로 인증 합니다.  
인증이 성공하면 유저가 ‘k get po -A’라는 명령을 수행할 권한이 있는지 클러스터 롤 바인딩과 롤 바인딩을 검사 합니다.  
인증과 인가까지 통과한 요청은 어드미션 컨트롤러의 뮤테이팅Mutating 어드미션Admission과 밸리데이팅Validating 어드미션Admission 과정까지 통과해야 합니다.  
뮤테이팅 어드미션은 요청을 정책에 맞게 변형하는 과정이고 밸리데이팅 어드미션은 최종적으로 요청을 수락할 지 판단하는 과정입니다.  
예를 들어 뮤테이팅 어드미션에서 무조건 현재 네임스페이스의 파드 리스트만 리턴 하도록 요청을 변형 하였다면 전체 네임스페이스의 파드를 볼 수 있는 권한이 있더라도 현재 네임스페이스의 파드 리스트만 리턴 됩니다.  
밸리데이팅 어드미션에서 ‘Guest’그룹의 사용자는 파드 리스트를 못 보게 정의 하였고 현재 사용자가 ‘Guest’그룹에 속한다면 최종적으로 요청은 거부 됩니다.

![[../../resources/Pasted image 20230613230056.png|500]]


## KubeConfig

```
k get node --kubeconfig config
```
로 k get node를 할때 Kubeapiserver 로 요청을 보낼텐데 이때 authentication을 통해 인증메커니즘을 거친다. 이때 이런 옵션들을 지정해주는 건 매우 힘든 작업일 것이다. 
따라서 kubeconfig 파일에 이런 값들을 구성하고 kubectl은 여기서 값을 가져와 자동으로 지정해주는 것이다. 

eks 접속
```
[current context 변경]
aws eks update-kubeconfig --region ap-northeast-2 --name nudlsdapne2-beia
or 
kubectl config use-context username@clustername

kubectl config view
> config 확인 가능
or
cat ~ /.kube/config

# context 조회
kubectl config get-contexts

```


kubeconfig 기본 홈에 있는 컨피그가 아닌 다른 config파일로 적용하고 싶을때

```
kubectl config --kubeconfig=/root/my-kube-config use-context research
```
> my-kube-config라는 config 파일에서 current-context를 research라는 name인 context로 바꾼다. 

```
kubectl config --kubeconfig=/root/my-kube-config current-context
```
my-kube-config라는 config 파일에서 current-context 조회

#### kubectl이 어느 클러스터에 연결되어있는지 조회

```
kubectl config current-context
```

### default config 파일 다른 config 파일로 변경
```
ex)
mv /root/my-kube-config /root/.kube/config
mv ~/my-kube-config ~/.kube/config
```

### KubeConfig File
$Home/.kube/config 
yaml으로 되어있다.
세개의 영역으로 나뉜다. 
- cluster
  어느 클러스터에 접근하는지
- context
  user@cluster로 user와 cluster를 mapping해주는것이다. 
- user
  어느 user에게 권한이 있는지

context에 
namespace를 지정할 수도 있다. 
```
contexts:
  - name : 
    context:
      cluster:
      user:
      namespace:
```


## **API Groups**
![[../../resources/Pasted image 20230616221749.png|500]]
![[../../resources/Pasted image 20230616221816.png]]

## Authorization

Node
  노드 권한 부여는 kubelets에서 수행한 API 요청을 특별히 권한 부여하는 특수 목적 권한 부여 모드
  노드 권한 부여자는 kubelet이 API 작업을 수행할 수 있도록 한다.
  노드 권한 부여자는 kubelet이 올바르게 작동하는 데 필요한 최소한의 권한 집합을 갖도록 권한을 추가하거나 제거할 수 있다.
ABAC (Attribute Base Access Control)
  json format으로, user or group에 permission attribute을 설정한다.
  개별 유저 or 그룹마다 권한을 설정해주어야 하므로 관리하기가 까다롭다. 
  새로운 유저가 들어와 수정하게 되면, kube-api를 재시작해 update해줘야한다.
RBAC (Role Base Access Control)
  ABAC과 다르게 rule를 먼저 만들어 두고 user or group에 mapping 하는 방법이다. rule 수정이 필요한 경우 kube-api를 재시작하지 않고 reflect만 하면된다.
  https://arisu1000.tistory.com/27848
Webhook 
  인증메카니즘을 외부에 위탁하고싶다면
  http 콜백으로 어떤 event가 발생할때마다 http post가 발생한다. 
  주로 third party module과 함께 사용한다. 
AlwaysAllow
AlwaysDeny

이는 kube API의 Authorization 모드의 옵션을 통해 설정한다. 
아무설정을 안한다면 AlwaysAllow이다. 
multiple로도 가능하다.



### 해당 명령을 할 수 있는지 체크 방법
```
kubectl auth can-i (command) --as (user name) --namespace (ns name)
kubectl auth can-i create deployment
kubectl auth can-i delete nodes
kubectl auth can-i create deployment --as dev-user --namespace test
```

![[../../resources/Pasted image 20230616225224.png]]
coreRole인 경우 apiGroup을 ""로 한다. 
`resourceNames` list. When specified, requests can be restricted to individual instances of a resource. Here is an example that restricts its subject to only `get` or `update` a [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) named `my-configmap`

![[../../resources/Pasted image 20230616225454.png]]


## **실습**

### authorization-mode 확인하기 

kube-apiserver settings 위치
```
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```
로 확인하기 
또는 
```
ps -aux | grep authorization
```

### dev-user라는 user가 get pod하는 권한이 있는지 확인하기 

 ```
 k auth can-i get pod --as dev-user
 ```
 no나와서 
 이걸로 확인했었는데
애초에 
```
k get pods --as dev-user
```
로 해봐도 되는거였다. 


# Cluster Role
role은 ns안에서 만들어져서 그 ns안에서만 사용가능하다.

### Namespaced
pods, replicaset, jobsm deployment, service, secrets, roles, rolebindings, configmaps, PVC
같은 리소스는 지정된 ns에 생성된다. 

### Cluster Scoped
nodes, PV, clusterroles, clusterrolebindings, certufucatesigningrequests, namespaces
의 클러스터 범위의 리소스들은 ns을 지정하지 않는다.  

이 목록들을 확인하고자 한다면 
```
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```

### *clusterroles*
클러스터 범위 리소스에서




