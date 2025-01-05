## DNS in kubernetes

![[Pasted image 20240616211005.png|500]]
Kubernetes는 클러스터를 설정할 때 default로 *built-in DNS서버*를 배포한다. 
(Kubernetes를 수동으로 설정하는 경우에는 직접 설정될것) 

테스트 파드에서 웹 서버에 액세스할 수 있도록 서비스를 생성하는데, 
서비스가 생성될 때마다 Kubernetes DNS는 서비스에 대한 레코드를 생성한다. 
서비스 이름을 IP 주소에 매핑하므로 클러스터 내에서 이제 모든 파드가 해당 서비스 이름을 사용하여 이 서비스에 도달할 수 있게 된다. 

<네임스페이스>
*네임스페이스 내의 모든 리소스는 이름으로만 서로를 부르고, 다른 네임스페이스에 있는 리소스를 부르려면 전체 이름을 사용한다.*
default 네임스페이스인 동일한 네임스페이스에 있다고 하면,
서비스 이름 web-service만 사용하여 테스트 파드에서 웹 서비스에 간단히 도달할 수 있다.

![[Pasted image 20240616210744.png|500]]
웹 서비스가 Apps라는 별도의 네임스페이스에 있다고 가정

default 네임스페이스에서 참조하려면 `web-service.apps`
Web Service: 서비스의 이름 
Apps: 네임스페이스의 이름 

- *각 namespace에 대해 DNS 서버는 하위 도메인을 만든다.* 
  따라서 네임스페이스에 대한 모든 파드 및 서비스는 네임스페이스 이름의 하위 도메인 내에서 함께 그룹화된다.

- *모든 서비스는 SVC라는 또 다른 하위 도메인으로 함께 그룹화된다.*
  `webservice.apps.svc` 라는 이름으로 애플리케이션에 연결할 수 있다.

- *모든 서비스와 파드은 default로 cluster.local로 설정된 클러스터의 경로 도메인으로 함께 그룹화 된다.*
  `webservice.apps.svc.cluster.local` 사용하여 서비스에 액세스할 수 있으며 이는 서비스의 정규화된 도메인 이름이다.

![[Pasted image 20240616210803.png|500]]

 파드는 어떨까? 
 
- 파드에 대한 레코드는 default로 생성되지 않지만 명시적으로 활성화할 수 있다. 
- 활성화되면 파드에 대한 레코드도 생성되는데, 파드 이름을 사용하지 않는다.
- 각 파드에 대해 Kubernetes는 IP 주소의 점을 대시로 대체하여 이름을 생성하며, namespace은 동일하게 유지되고 유형은 pod로 설정된다. root domain은 항상 cluster.local이다.

## CoreDNS in kubernetes

![[Pasted image 20240616212502.png|500]]

2개의 IP 주소가 있는 2개의 파드이 주어졌다고 가정했을때
서로를 해결하는 쉬운 방법은 각각의 etc 호스트 파일에 항목을 추가하는 것이다.
물론 클러스터에 수천 개의 파드가 있고 매분 수백 개가 생성 및 삭제되는 경우 이는 적합한 솔루션이 아니다.

따라서 이러한 항목을 *중앙 DNS 서버*로 이동시킨다. 
-> etc/resolv.conf 파일에 nameserver 서버가 DNS 서버의 IP 주소(10.96.0.10)에 있음을 지정한다. 
새 파드가 생성될 때마다 다른 파드가 새 파드에 액세스하고 
파드의 etc/resolv.conf 파일이 DNS 서버를 가리키도록 구성할 수 있도록 해당 파드의 DNS 서버에 레코드를 추가한다.

이전 DNS 내용과 같이
파드의 경우 파드의 IP 주소에서 점을 대시(-)로 대체하여 호스트 이름을 구성하여 DNS를 구현한다.

### CoreDNS
클러스터 내에 DNS 서버를 배포하는데,
버전 1.12 이전: Kube DNS
Kubernetes 버전 1.12 이후: 권장 DNS 서버는 *CoreDNS*

![[Pasted image 20240616213659.png|500]]
그렇다면 클러스터에서 CoreDNS는 어떻게 설정될까? 

- coreDNS 서버는 Kubernetes 클러스터의 `kube-system` namespace pod로 배포된다.
- replicas set으로 복제본 존재(deployment)
-  CoreDNS에는 *configuration 파일* 이 필요하다. `Corefile`->  `/etc/coredns` 에 위치하며, 파일에는 여러 *플러그인*이 구성되어 있다.

#### Corefile
- 플러그인으로는 오류 처리, health , 메트릭 모니터링, cache 등으로 구성되며,
- CoreDNS가 Kubernetes와 작동하도록 하는 플러그인은 *Kubernetes 플러그인*
- 클러스터의 *최상위 도메인 이름*이 설정되는 곳으로 cluster.local 임을 볼 수 있다.
  따라서 CoreDNS서버의 모든 레코드는 이 도메인에하위에 속하게 된다.
- proxy etc/resolv.conf
  예를 들어 파드가 www.google.com에 도달하고자 할때, DNS 서버가 해결할 수 없는 레코드는 resolv.conf 파일에 지정된 nameserver로 전달되는 것처럼 etc/resolv.conf 파일은 Kubernetes 노드에서 nameserver로 사용된다.
- Corefile은 configmap object로 파드에 전달된다. 
   configuration을 수정해야 하는 경우, configmap object로 편집할 수 있다. 
---
![[Pasted image 20240616213723.png|500]]

파드는 DNS 서버에 도달하기 위해 어떤 주소를 사용할까? 
CoreDNS 솔루션을 배포할 때 클러스터 내의 다른 컴포넌트에서 사용할 수 있도록 서비스도 생성하는데,
서비스 이름은 default로 *kube-dns*로 지정된다. 
이 서비스의 IP 주소는 파드에서 nameserver로 구성되며, 
직접 구성할 필요없이, 파드의 DNS config은 파드가 생성될 때 Kubernetes에 의해 자동으로 수행된다.

이를 담당하는 것은 바로 *kubelet* 
kubelet의 config 파일을 보면 그 안에 DNS 서버와 도메인의 IP를 확인 할 수 있다.

![[Pasted image 20240616213825.png|400]]
![[Pasted image 20240616213832.png|400]]
파드가 올바른 nameserver로 config되면 이제 다른 파드 및 서비스를 확인할 수 있게 되며 
위 이미지처럼 도메인으로 DNS를 통해 다른 파드및 서비스에 액세스할 수 있다.

nslookup 또는 host 명령을 사용하여 웹 서비스를 수동으로 조회하려고 하면 
웹 서비스의 정규화된 도메인 이름(web-service.default.svc.cluster.local)이 반환된다. 

하지만 `hoswt web-service` 라고만 했는데 전체 이름은 어떻게 찾았을까? 
![[Pasted image 20240616223437.png|400]]
resolv.conf 파일에 default.service.cluster.local, svc.cluster.local 및 cluster.local 로 설정된 search 항목이 있기 때문이다.
그러나 서비스에 대한 search 항목만 있으므로 동일한 방식으로 포드에 연결할 수 없ek.
파드는 host 질의하기 위해서는 포드의 전체 도메인 지정해야 한다.


## Ingress

![[Pasted image 20240616225626.png|400]]
외부에서 애플리케이션에 access 하기 위해서.
nodeport를 사용 할 수 있는데.
이는 사용자가 node의 ip와 3만대의 어려운 port번호를 기억해야 가능하다.

![[Pasted image 20240616225644.png|400]]
그럴 수는 없으니,
노드의 IP를 가리키도록 *DNS 서버*  구성과 port를 기본적인 포트로 사용하기 위해 *proxy-server*를 구성하여
도메인으로 접속하도록 할 수도 있을 것이다.
![[Pasted image 20240616225716.png|400]]
이에 더불어 GCP 같은 퍼블릭 클라우드 환경과 함께라면 어떨까.
Nodeport type 대신에 *로드밸런서* type으로 구성할 수 있게 된다.
포트를 프로비저닝하는 것은 계속 수행하면서,
Kubernetes는 이 서비스에 대한 네트워크 로드 밸런서를 프로비저닝하기 위해 GCP에 요청을 보내며,
요청을 받으면 GCP는 트래픽을 모든 노드의 서비스 포트로 라우팅하도록 구성된 로드 밸런서를 자동으로 배포하고, 
해당 정보를 Kubernetes에 반환한다. 
로드 밸런서에는 애플리케이션에 액세스하기 위해 사용자에게 제공할 수 있는 외부 IP가 있으며. 
이 경우에는 DNS가 이 IP를 가리키도록 설정하고 사용자는 URL myonlinestore.com을 사용하여 애플리케이션에 액세스하면 되는 것이다.

![[Pasted image 20240616225734.png|500]]
하지만 위와 같이 사이즈가 커지면서 로드밸런서가 많아지게 되면?
 로드 밸런서 각각에 대해 비용을 지불해야 하며 이러한 로드 밸런서가 많으면 클라우드 청구서에 역으로 영향을 미칠 수 있다.
*그렇다면 사용자가 입력하는 URL을 기반으로 각 로드 밸런서 간에 어떻게 트래픽을 전달할까?*
*URL을 기반*으로 트래픽을 다른 서비스로 리디렉션할 수 있는 또 다른 프록시 또는 로드 밸런서가 필요하다.

Kubernetes 클러스터 내에서 모든 것을 관리하고 
나머지 애플리케이션 deployment 파일과 함께 존재하는 또 다른 Kubernetes definition 파일로 모든 configuration을 가질 수 있다면 좋지 않을까?

![[Pasted image 20240616225749.png|500]]
*이것이 Ingress가 필요한 이유이다.*
Ingress는 사용자가 *URL 경로*를 기반으로 클러스터 내의 다른 서비스로 라우팅하도록 구성할 수 있는 외부에서 액세스 가능한 단일 URL을 사용하여 애플리케이션에 액세스할 수 있도록 도와준다. 
동시에 SSL 보안도 구현할 수 있다.

*간단히 말해 Ingress를 Kubernetes 클러스터에 built-in된 레이어 7 로드 밸런서라고 생각할 수 있다.*

이제 Ingress를 사용하더라도 클러스터 외부에서 액세스할 수 있도록 expose 해야한다. 
따라서 여전히 노드 포트로 게시하거나 cloud native 로드 밸런서를 사용하여 publish 해야 하지만 이는 일회성 configration일 뿐이다.

#### ingress 구성방법
![[Pasted image 20240616225807.png|500]]
앞으로 Ingress controller 에서 모든 로드 밸런싱 authentication, SSL, URL 기반 라우팅 configuration을 수행할 수 있다.
#### Ingress controller
- reverse 프록시나 NGINX, HAProxy 또는 Traefik과 같은 로드 밸런싱 솔루션을 사용할 수 있다.
- 이것들을 Kubernetes 클러스터에 배포하고 트래픽을 다른 서비스로 라우팅하도록 구성한다. 
- configuration에는 URL route 정의, SSL 인증서 configuration 등이 포함될 것이다.

여기에 나열된 솔루션 중 하나인 지원 솔루션을 배포한 다음, rule 집합을 지정하여 Ingress를 구성하면되는데, 이때
배포하는 솔루션 -> *Ingress controller*
구성하는 rule 집합 ->  *Ingress resource*

#### Ingress resource
ingress resource 는 pod, deployment 및 서비스를 생성하는 데 사용하는 것과 같은 definition 파일을 사용하여 생성된다.

(주의)
Kubernetes 클러스터는 default로 Ingress controller 함께 제공되지 않는다는 점 기억
이 과정의 데모에 따라 클러스터를 설정하면 Ingress controller builtin되어 있지 않으니 주의

*즉,  ingress는 말 그대로 규칙들의 모음이며, 이러한 요청을 실제로 처리하는 것은 ingress controller*
### Ingress Controller
![[Pasted image 20240616235253.png|400]]
Ingress controller에 사용할 수 있는 솔루션은 여러 가지가 있다.

nginx를 예시로 들어보자면,
이때 nginx는 일반 nginx가 아닌 쿠버네티스 ingress controller로 사용되도록 제작된 특수 nginx이다.
단순 로드밸런서만 하는 것이 아니며, ingress 리소스에 대해 kubernetes 클러스터를 모니터링하고 이에따라 nginx서버를 구성하기 위한 추가 intelligence가 built-in 되어있다.

![[Pasted image 20240616235513.png|600]]
`Kind: Deployment` 로 리소스를 생성.
`nginx-ingress` : label 지정 
#### image, args
`image`: nginx-ingress-controller
쿠버네티스 ingress controller로 사용되도록 제작된 *특수 nginx로 고유한 요구 사항*이 존재한다.
이미지 내에서 nginx 프로그램은 nginx ingress controller 위치에 저장되므로 nginx controller 서비스를 시작하기 위한 것들을 *커맨드*로 전달해야한다.

nginx를 사용할때
path to store the logs, 
keep alive threshold, 
SSL settings, 
session timeouts 등과 같은 일련의 configuration 옵션이 존재하는데

nginx controller 이미지에서 이러한 configuration 데이터를 분리하기 위해
*config map* object를 생성하고 이를 전달한다. 

#### env, ports
파드 이름과 네임스페이스를 포함하는 두 개의 환경 변수를 전달해야 한다.
ingress controller로 사용하는 포트(80 및 443)를 지정

#### NodePort
![[Pasted image 20240616235535.png|600]]
- selector: ingress controller를 외부로 expose 할 필요가 있으므로 위 deployment에 연결하기 위해 *selector*로 위 deployment label을 지정. 
- type: NodePort 

#### service account
 ![[Pasted image 20240616235553.png|500]]
Ingress controller에는 Kubernetes 클러스터에서 Ingress 리소스를 모니터링하고 
변경 사항이 있을 때 nginx 서버를 구성하기 위한 추가 intelligence가 built-in 되어 있는데,
ingress 컨트롤러가 이를 수행하려면 올바른 권한 집합이 있는 서비스 계정이 필요하다.
이를 위해 올바른 role과 role-binding 으로 서비스 계정을 만들어야한다.

![[Pasted image 20240616235609.png|500]]
### Ingress Resource

ingress 리소스는 ingress controller에 적용되는 rule 및 configuration 집합이다.
들어오는 모든 트래픽을 URL을 기반으로 트래픽을 다른 애플리케이션으로 라우팅하도록 rule을 configuration할 수 있다. 
따라서 사용자가 myonlinestore.com/wear로 이동하면 애플리케이션 중 하나로 라우팅하거나 
사용자가 watch *URL*을 방문하면 비디오 앱으로 라우팅하는 등의 작업을 수행한다. 
또는 *도메인 이름 자체*를 기반으로 사용자를 라우팅할 수 있다. 

#### ingress resource 생성.
![[Pasted image 20240617001049.png|600]]
`apiVersion: extension/v1beta1`
`kind: Ingress`
`backend`: 트래픽이 라우팅되는 위치 정의
단일 백엔드인 경우 실제 rule이 존재하지 않으므로 backend 색션에 서비스이름과 포트만 지정.

#### path 기반
![[Pasted image 20240617001103.png|600]]
path기반인 경우
rules의 
http.paths 하단에 path를 각각 지정하고 그다음 backend를 지정하여
path별 라우팅을 할 수 있다.

![[Pasted image 20240617001112.png|500]]

#### Domain 기반 라우팅
![[Pasted image 20240617001127.png]]
path 가 아닌 애초에 도메인으로 라우팅을 구별하고 싶다면,
rules.host 에 도메인을 각각 지정하여
rule을 생성하면된다.

### Changes
![[Pasted image 20240617001855.png|600]]

Now, in k8s version **1.20+** we can create an Ingress resource from the imperative way like this:-
`kubectl create ingress <ingress-name> --rule="host/path=service:port`
**Example 
`kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80`

### rewrite-target 옵션
![[Pasted image 20240617002332.png|500]]
rewrite-target은 ingress에 정의된 경로로 들어오는 요청을 설정된 경로로 전달한다.
-> This rewrites the URL by replacing whatever is under `rules->http->paths->path`

`/pay` in this case with the value in `rewrite-target`. 
This works just like a search and replace function.

For example: `replace(path, rewrite-target)`  
In our case: `replace("/path","/")`

![[Pasted image 20240617002726.png|500]]
rewrite-target: /$2의 의미는 
/something으로 요청되었을 때 regex의 두번째 그룹으로 요청을 보내겠다는 의미. 
만약 /something/color로 요청한다면 /color로, something/color/red라면 color/red로 요청이 전달