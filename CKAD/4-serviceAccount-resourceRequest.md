## **Service Account**
쿠버네티스 계정에는 두가지가 있다. 
user account, service account
user는 사용자가 사용하는 계정
service는 컴퓨터가 사용하는 계정이다. 

user- management 하기 위해 클러스터에 access 관리자나 응용프로그램 개발자가 배포를 위해 access를 예로 들 수 있다. 
sevice- 앱이 쿠버네티스 클러스터와 상호작용할때, 
프로메테우스 같은 모니터링 앱이 service account 를 통해 쿠버네테스 api를 퍼포먼스 메트릭으로 끌어오는 등 
젠킨스 같은 자동화된 빌드툴이 클러스터에 앱을 배포하는 데 service account 사용

![[../../resources/Pasted image 20230603161043.png|400]]
~~When the service account is created, it also creates a token automatically.~~

~~The service account token is what must be used by the external application while authenticating to the Kubernetes API. 
The token, however, is stored as a *secret object*. In this case its named dashboard-sa-token-kbbdm.~~
버전 바뀌면서 토큰 자동생성 안되게 바뀜.
`kubectl create token dashboard-sa`
를 해야지만 토큰이 생성된다. 


![[../../resources/Pasted image 20230603161206.png|500]]
The secret object is then linked to the service account. To view the token, view the secret object by running the command kubectl describe secret.

![[../../resources/Pasted image 20230603161331.png|500]]
This token can then be used as an authentication bearer token while making a rest call to the kubernetes API. For example in this simple example using curl you could provide the bearer

![[../../resources/Pasted image 20230603161511.png]]
But what if your third party application is *hosted* on the kubernetes cluster itself. For example, we can have our custom-kubernetes-dashboard or the Prometheus application used to monitor kubernetes, deployed on the kubernetes cluster itself.

In that case, this whole process of exporting the service account token and configuring the third party application to use it can be made simple by automatically *mounting the service token secret as a volume inside the POD hosting the third party application.* That way the token to access the kubernetes API is already placed inside the POD and can be easily read by the application.

![[../../resources/Pasted image 20230603163122.png|500]]
쿠버네티스 안의 모든 namespace에는 default service account가 create되어있다. 
Each namespace has its own default service account.
~~Whenever a POD is created the default service account and its token are automatically mounted to that POD as a volume mount.~~ *버전 1.22부터 tokenrequestapi를 통해 token 이 발급되어 수명을 가진 token 으로 변경 위의 사진의 pod 정의와 지금은 다를 것이다*
```yaml
volumes:
  projected:
    defaultMode: 420
    sources:
    -  serviceAccountToken:
         expirationSeconds: 3607
         path: token
```
이 형식과 같다. 

The secret token is mounted at location /var/run/secrets/kubernetes.io/serviceaccount inside the pod. So from inside the pod if you run the ls command

![[../../resources/Pasted image 20230603170753.png|500]]
efault service account is very much restricted. It only has permission to run basic kubernetes API queries.

Remember, *you cannot edit the service account of an existing pod, so you must delete and re-create the pod.* However in case of a *deployment*, you will be able to edit the serviceAccount, as any changes to the pod definition will automatically trigger a new roll-out for the deployment. 

![[../../resources/Pasted image 20230603172238.png|400]]
자동으로 default service account 가 mount안되도록 하고 싶을때 

### kubernete
### v1.22
*TokenRequestAPI*
service account token의 안정성과 확장성을 높여주는 메커니즘 도입하여 쿠버네티스 service account token을 프로비전하는데 사용됐다. 

tokenrequest api에서 발급한 token은 
Audience bound 사용자바운드
time bound
object bound
가 된다. more secure하다.

이버전부터 pod가 생성되면 위에서 계속 본 token에 의존하지 않고 
대신에 tokenrequest api를 통해 정해진 **수명을 가진 token** 이 service account approve controller에 의해 생성된다.
*pod가 생성되면 이 token은 volumes로 pod에 mount된다.*

### v1.24
secret token 축소사항 
`kubectl create serviceaccount (name)`
으로 서비스계정을 만들때 자동으로 토큰이 생성되지 않고 
`kubectl create token dashboard-sa`
로 토큰생성 명령을 해야 토큰이 생성된다. 

이렇게 생성된 토큰을 복사해서 해독하면 
유효기간이 정의되어있음을 볼 수 있다. 
`"exp": 16634234234`
시간제한을 지정하지 않았다면 명령어를 친지 1시간 후이다. 
명령에 추가옵션을 전달해 토큰의 유효기간을 늘릴 수도 있다. 

만약 옛날 버전으로 secret을 만들고 싶다면
secret yaml을 생성시
```yaml
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata: 
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: dashboard-sa
``` 
sevice account먼저 생성후 secret생성한다. 
이렇게 옛날버전 서비스계정의 secret token을 생성하면 만료되지 않는 토큰을 사용할 수 있다. 

### service account token secrets
tokenrequest api를 사용하지 않으면 서비스 계정 토큰 secret을 만들 수 없다고 되어있다. 

위에서 kubectl 명령어로 생성한 토큰 명령어(kubectl create token) command to obtain a token from tokenrequest api
하거나 
옛날버전으로 토큰생성해서 만료하지 않는 토큰을 생성하는 방법이 있다 .

## **ResourceRequest**

![[../../resources/Pasted image 20230603184946.png]]
*kubernets scheduler*는 pod 배치시 필요한 cpu, mem을 고려해서 node에 배치한다. 이런 pod마다 필요한 resource가 다를텐데 이를 *resource request*라 한다. 
자원이 없을 경우 위와 같은 에러이벤트가 발생할 것이다 

![[../../resources/Pasted image 20230603185144.png]]
위와 같이 containers에 resources를 정의해주면 된다. 

![[../../resources/Pasted image 20230603185253.png|500]]
*1 count of CPU is equivalent to 1 vCPU.* That’s 1 vCPU in AWS, or 1 Core in GCP or Azure or 1 Hyperthread

![[../../resources/Pasted image 20230603185401.png|500]]

![[../../resources/Pasted image 20230603192133.png|500]]
request, limit이 있을때 
처음 pod를 만들때 이 요청과 제한은 없다. 정의를 해주어야 요청과 제한을 줄 수 있는 것이다. 

*request는 정의한 값만큼의 리소스를 보장해주는것이고 
limit는 말그대로 제한이다.*

limit이 정해졌다 했을때 
cpu는 이를 초과할 수 없다. memory는 이를 초과할 수 있으나 반복적으로 이를 초과한다면 *OOM(out of memory)* 에러가 발생하여 메모리에 저장된게 다 죽어버리는 현상이 생긴다. memory를 지워야 memory가 생긴다는 원리다. 
limit을 주지 않은 경우 limit이 초과하여 다른 pod의 리소스도 침범해 다른 pod도 죽여버리는 경우가 생길 수 도 있다. 
그런데 *limit을 제한 두고 싶지 않는 경우가 있을 수 있다. 이때 모든 pod에 request가 모두 정의되어있다고 하면 limit을 초과해도 다른 pod를 죽여버리는 건 예방할 수 있다.*

`kind: LimitRange`
로 resource limit을  yaml으로 정의해 생성할 수 도 있다. 

만약 namespace 단위로 리소스를 제한하고 싶다면 
`kind: ResourceQuota`
를 생성하여 지정할 수 도 있다. 
