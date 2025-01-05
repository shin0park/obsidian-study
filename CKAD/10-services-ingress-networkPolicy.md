![[../../resources/Pasted image 20230609201534.png|444]]
service:
애플리케이션 안 밖의 다양한 구성요소들간의 통신을 가능하게한다. 
애플리케이션을 다른 애플리케이션 또는 사용자와 연결하는데 도움을 준다. 

### NodePort
서비스가 노드의 포트에 내부 포트를 엑세스 하게하는 것
노드의 포트와 포드의 포트를 매핑한다. 
서비스는 사실상 노드안의 가상 서버와 같다. 
1. targetPort
   웹서버라하면 실제 pod에 떠있는 웹서버의 port이다. 웹서버라면 80이 기본포트일 것이다. 결국 사용자가 접속해야되는 것은 여기로 target port라한다. 
2. Port
   서비스의 port이다. 서비스도 사실상 노드의 가상서버이다. 따라서 서비스의 cluster ip도 존재해 ip를 갖고있다. 
3. nodePort
   노드의 포트이다. 기본적으로 노드포트는 *30000 ~ 32767* 유효범위가 존재한다.  

### ClusterIP
클러스터 내부에 가상 ip를 만들어 다양한 서비스간의 통신을 가능하게 한다. (예를 들면 프론트엔드 서비스와 백엔드 서비스)
![[../../resources/Pasted image 20230610000015.png|400]]
보통 이렇게 파드들이 계층간 역할로 나뉘어 있을것이다. 이 계층간의 통신이 필요하다. 
포드는 모두 ip주소가 할당되어있다. 이 ip 주소는 고정적이지 않다. 포드의 삭제 생성이 자주일어나기 때문이다. 앱간의 내부 ip에만 의존할 수가 없다.
따라서 *서비스를 통해 포드들을 하나로 묶고 인터페이스를 통해 단체 포드에 접속할 수 있게 한다.*  
즉 백엔드 역할인 포드는 모두 하나로 묶고 여기에 접속 할 수 있는 단일 인터페이스를 제공하는 것이다. 
각 계층은 필요한 대로 확장 또는 이동 할 수 있다. 다양한 서비스간의 통신에 영향주지 않는다. 
*각 서비스는 ip를 갖고 클러스터 내부에 할당된다. 다른 포드가 서비스에 접근해서 통신한다. 이런 유형의 서비스를 클러스터IP라 한다.**

### LoadBalancer
application을 provisioning한다. 
예를 들어 프론트엔드의 다양한 웹서버에 로드를 배포하는것이 있다.

### create service
pod와 동일하게 정의해서 생성하면된다. 
```
apiVersion: v1
kind: Service
metadata: 
  name: myapp-service
spec:
  type: NodePort
  ports:
  - targetPort: 80
    port: 80
    nodePort: 30008
  selector:
    app: myapp
    type: front-end
```
ports는 배열이며
*targetport를 지정하지 않으면 포트와 동일하게 인식된다.*
노드포트를 제공하지 않으면 유효한 범위의 30000~32767사이에서 자동으로 포트가 할당된다. 
pod와 service 간의 연결은 *label-selector* 로 연결하면 된다. 
kube controller를 통해 서비스를 생성한다.

만약 포드가 여러개라면?
서비스의 selector에 정의된 label을 
생성할 포드의 label에 넣으면 다른 조치를 할 필요가 없다. 
그리고 서비스는 내장된 load balancer 로 파드에 부하분산시킨다. 

노드안에서가 아닌 다중노드에서 포드 분산인 경우에는? 
이경우에는 클러스터 내의 개별 노드포트에 웹응용프로그램이 있다. 
추가적인 구성없이 서비스 생성할때 쿠버네티스는 자동으로 클러스터 내 모든 노드를 가로질러 목표 포트를 클러스터 내 모든 노드에 같은 포트로 매핑한다. 

**즉 단일 노드의 다중포드든 다중노드 위의 다중포드든 상관없이 서비스는 똑같이 생성된다. 
서비스 생성단계동안 추가 단계를 밟지 않아도 된다.**
포드가 일부 망가지거나 업데이트되도 서비스는 자동으로 업데이트되어 매우 유연하게 적응한다. 
따라서 클러스터에 있는 노드의 IP와 node port번호를 사용해서 외부에서 접근할 수 있다. 

### create clusterip
![[../../resources/Pasted image 20230610000619.png|500]]

### Endpoint

		service
pod1 pod2 pod3

가 있다고 하자. 
포드를 만들때 label을 붙일텐데 예를 들어 app:fe라면 service를 이 포드에 붙이기 위해선 selector도 app:fe로 해야한다. 
서비스는 같은 레이블로 모든 포드를 식별하고 그 포드에 트래픽을 전달한다. 우선 이렇게 레이블로 포드를 식별한 다음 
서비스가 endpoint를 갖게된다. 서비스의 endpoint는 각각의 pod라 볼 수 있다. 

## **Ingress**

![[../../resources/Pasted image 20230611144501.png|333]]
wear라는 app 이 있고 mysql pod와 연결하고 있는 상태이다. mysql 파드를 하나로 묶어서 하나의 ip로 접근하는 clusterip 서비스가 만들어져있는 상태이고 wear에 nodeport 서비스가 붙어 38080포트로 외부와 접속할 수 있게 되어있다. 
하지만 사용자가 매번 ip 주소와 포트번호로 접속하기에 번거롭다. 
**DNS**서버에 도메인을 생성하고 **proxy-server**를 통해 기본 포트 80으로 접속해도 38080으로 연결되게 아래의 구조처럼 형성되게 한다.
![[../../resources/Pasted image 20230611144524.png|333]]
도메인 url만 쳐도 응용프로그램에 접속할 수 있게 된다.
위의 이미지는 온프레미스인 경우이다.

*퍼블릭 클라우드* 환경이라 생각해보자.
In that case, instead of creating a service of type NodePort for your wear application, you could set it to type *LoadBalancer*.
nodeport의 여하린 provision a high port 뿐만아니라 native load balancer 역할도 한다.
The LoadBalancer has an external IP that can be provided to users to access the application. In this case we set the DNS to point to this IP and users access the application using the URL.

![[../../resources/Pasted image 20230611151859.png|444]]
On receiving the request, GCP would then automatically deploy a *LoadBalancer* configured to route traffic to the service ports on all the nodes and return its information to kubernetes. The LoadBalancer has an *external IP* that can be provided to users to access the application. In this case we set the DNS to point to this IP and users access the application using the URL my-online-store.com.

![[../../resources/Pasted image 20230611152013.png|555]]
새로운 application을 배포한다면 위에서 설명한 구조를 한세트 더 만들어야될 것이다. 
The new load balancer has a new IP. 
Remember you must pay for each of these load balancers and having many such load balancers can inversely affect your cloud bill.

![[../../resources/Pasted image 20230611152140.png|555]]
You need yet another *proxy or load balancer* that can re-direct traffic based on URLs to the different services. Every time you introduce a new service you have to re-configure the load balancer.
And finally you also need to enable *SSL* for your applications, so your users can access your application using https.

이렇게 application 이 새로 생성될때마다 이걸 모두 만들어주고 관리하는건 매우 힘들것이다. 
이로서 **ingress**가 나오게 되었다. 
![[../../resources/Pasted image 20230611152402.png|444]]
Ingress helps your users access your application using a single Externally accessible URL, that you can configure to route to different services within your cluster based on the URL path, at the same time terminate TLS.
#### ingress as a layer 7 load balancer
you still have to either publish it as a NodePort or with a Cloud Native LoadBalancer.
But that is just a one time thing. Going forward you are going to perform all your load balancing, Auth, SSL and URL based routing configurations on the Ingress controller.

![[../../resources/Pasted image 20230611152710.png|444]]
쿠버네티스가 ingress를 수행한다. 먼저 지원되는 솔루션을 배포한다. 
그리고 그 인그레스 구성을 위한 rule 집합을 명시한다. 
여기서 배포하는 솔루션을 **ingress controller** 라고 하고 그리고 구성하는 rule 모임을 **ingress resource**라 한다. 인그레스 리소스는 definition 파일을 이용해 생성된다. 
*쿠버네티스 클러스터는 기본적으로 ingress controller 가 없다.* 만들어줘야 인그레스가 작동할 것이다. 

### **Ingress Controller**
인그레스에 대한 솔루션은 다양하다. 
![[../../resources/Pasted image 20230611153349.png|444]]
ingress controller는 another 로드벨런서나 nginx 서버가 아니다. 
로드벨런서와 nginx는 구성요소 그 일부일 뿐이다. 
ingress controller는 추가 intelligent가 내장되어있어 새로의 정의 혹은 리소스를 위한 쿠버네티스 클러스터를 *모니터링*하고 그에 따라 nginx 서버같은걸 구성하는 것이다. 

# create
## Ingress controller
![[../../resources/Pasted image 20230611160926.png|500]]
### args
*Within the image the nginx program is stored at location /nginx-ingress-controller. So you must pass that as the command to start the nginx-service.*
it has a set of configuration options such as the path to store the logs, keep-alive threshold, ssl settings, session timeout etc.  
### configMap
In order to decouple these configuration data from the nginx-controller image, you must create a ConfigMap object and pass that in. Now remember the ConfigMap object need not have any entries at this point. A blank object will do. But creating one makes it easy for you to modify a configuration setting in the future.
### env
You must also pass in two environment variables that carry the POD’s name and namespace it is deployed to. The nginx service requires these to read the configuration data from within the POD.
### port
ingress가 사용하는 port 번호 지정 

## ingress service
![[../../resources/Pasted image 20230611161244.png|555]]
이제 ingress controller를 외부에 노출 시킬 서비스가 필요하다. 

이런 작업들을 하려면 올바른 *권한 모음을 가진 service account*가 필요하다. 
올바른 role과 Rolebinding으로 Service Account를 만든다. 

## **Ingress Resource**
ingress controller에 적용된 규칙과 구성의 집합이다. 

![[../../resources/Pasted image 20230611143631.png]]
domain 이름이 다른경우 host도 추가. 
![[../../resources/Pasted image 20230611162416.png|555]]
이전 문법인거 참고 

## 실습
```
k get deploy -A
```
로 배포 목록을 보면 응용프로그램들 배포들과 ingress-controller 배포를 볼 수 있다. 각 배포가 들어있는 ns로 kgp를 하면 어떤 pod들이 올라가있는지 확인 할 수 있다. 

ingress resource는
```
k get ingress -A
```
로 조회할 수 있다.
그리고 그 리소스에 path로 clusterip인 서비스를 연결하게 될텐데 이 서비스들은 
```
k get service -A
```
로 조회할 수 있다. 

### ingress resource 생성
`k get service -n (namespace)`
로 연결할 서비스를 확인하고 
ingress를 생성해준다. 
`k create ingress -h`
로 help 를 볼 수 있다. 
```
k create ingress (ingress name) -n (ns) --rule="path"=(servicename):(port)
k create ingress ingress-pay -n critical-space --rule="/pay"=pay-service:8282
```

### FAQ - What is the rewrite-target option?

Different ingress controllers have different options that can be used to customise the way it works. NGINX Ingress controller has many options that can be seen [here](https://kubernetes.github.io/ingress-nginx/examples/). I would like to explain one such option that we will use in our labs. The [Rewrite](https://kubernetes.github.io/ingress-nginx/examples/rewrite/) target option.

Our `watch` app displays the video streaming webpage at 
`http://<watch-service>:<port>/`

Our `wear` app displays the apparel webpage at
`http://<wear-service>:<port>/`

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the /watch and /wear URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:  
  
`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/`

Without the `rewrite-target` option, this is what would happen:

`http://<ingress-service>:<ingress-port>/watch` --> `http://<watch-service>:<port>/watch`

`http://<ingress-service>:<ingress-port>/wear` --> `http://<wear-service>:<port>/wear`
Notice `watch` and `wear` at the end of the target URLs. The target applications are not configured with `/watch` or `/wear` paths. They are different applications built specifically for their purpose, so they don't expect `/watch` or `/wear` in the URLs. And as such the requests would fail and throw a `404` not found error.

  To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the `rewrite-target` option. This rewrites the URL by replacing whatever is under `rules->http->paths->path` which happens to be `/pay` in this case with the value in `rewrite-target`. This works just like a search and replace function.

For example: `replace(path, rewrite-target)`  
In our case: `replace("/path","/")`
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
	nginx.ingress.kubernetes.io/rewrite-target: /
  spec:
	rules:
	- http:
	  paths:
	  - path: /pay
	  backend:
		serviceName: pay-service
		servicePort: 8282
```


In another example given [here](https://kubernetes.github.io/ingress-nginx/examples/rewrite/), this could also be:
replace("/something(/|$)(.*)", "/$2")
![[../../resources/Pasted image 20230611172943.png|444]]


## **Network Policy**
우선 쿠버네티스는 클러스터 안의 모든 pod의 network policy에 대해선 **all allow**이다. 
여기서 만약 특정 pod에서만 ingress되게 하고 싶다면 network policy를 새성하면된다. 
![[../../resources/Pasted image 20230611215648.png|555]]
label - selector 조합으로 해당 policy를 적용할 파드를 정하고 ingress도 정한다. 
**네트워크의 중요한 특징인 들어오는 트래픽을 허용하면 그 트래픽에 대한 응답이나 회신이 자동으로 허용된다.** 따로 규정할 필요가 없다. 
만약 api pod에서 db pod로 ingress 를 db pod 에 뚫어줬다하자 그럼 그 응답은 따로 규정할 필요없이 응답이 가는건 맞는데 그렇다고 db pod에서 api pod를 부를 수 있다는 건아니다. 
그건 db pod의 egress도 뚫려있어야 한다. 이 두가지 차이점을 확실히 해야한다. 

만약 다른 ns 에 api pod가 있는 경우는 어떻게 될까?
prod ns 에 있는 name:api-pod 라벨을 가진 pod도 db 파드에 접근하도록 하고 싶다면 
namespaceSeletor라는 속성을 추가해서 ns를 추가한다. 단 해당 ns가 해당 라벨을 갖고있어야만 한다. 

![[../../resources/Pasted image 20230611221032.png|555]]
만약 위의 사진처럼 ns seletor만 설정한다면?
name:prod라는 라벨을 가진 ns에 있는 모든 pod는 db pod에 접근할 수 있는 것이다. 
단 db pod와 같은 ns는 해당 라벨을 가진 ns가 아니기때문에 db pod에 접근할 수 없다. 

![[../../resources/Pasted image 20230611220736.png|555]]
여기서 만약 namespaceSeletor앞에 -가 없이 podSeletor에 포함이면 *and*로 작동되고 -가 붙으면 *or*로 작동된다. 


![[../../resources/Pasted image 20230611221341.png|555]]
이렇게 특정 ip로 ingress를 설정 할 수도 있다. 
위처럼 - 가 붙어있으니
첫번째 정책은 같은 ns에 있는 api pod가 접근할 수 있다. 
두번째 정책은 prod라는 ns 에있는 모든 pod는 접근할 수 있다. 가 된다.

![[../../resources/Pasted image 20230611221923.png]] 