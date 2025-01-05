
### Security Primitives

host 에 대한 모든 엑세스는 보안되어야 한다.
- password based authentication *disabled*
- SSH Key based authentication

#### Secure Kubernetes
**kube-apiserver**
-> 쿠버네티스의 모든작업의 중심에는 kube-apiserver가 존재한다. 따라서 API 서버 자체에 대한 엑세스를 제어하는것이 1차 방어선이라 할 수 있다.
- Who can access? *Authentication*
	- *인증* 메커니즘에 의해 정의된다. API 서버에 인증하는 방법은 여러가지가 존재한다.
		- Files - Username and Passwords 
		- Files - Username and Tokens
		- Certificates
		- External Authentication providers - LDAP
		- Service Accounts  
		  -> 사용자 아이디와 암호로시작해 정적 파일, 토큰, 인증서에 저장할 수 있으며 LDAP 같은 외부인증공급자 와의 통합도 가능하다.
- What can they do? *Authorization*
	- 일단 클러스터에 엑세스 권한을 얻으면(인증) *인가* 메커니즘으로 권한을 부여한다.
		- RBAC Authorization
		- ABAC Authorization
		- Node Authorization
		- Webhook Mode


![[Pasted image 20240506141454.png|500]]
위 이미지의 쿠버네티스 클러스터와의 모든 통신 구성들은 TLS 암호화로 보안된다.

Network Policy - 클러스터내의 파드간의 통신은 네트워크 정책을 통해 엑세스 권한을 제어할 수 있다.
by default, 모든 파드는 클러스터내의 모든 파드에 접근할 수 있다.

### Authentication
![[Pasted image 20240506143844.png|500]]
Admins, Developers, End Users, Bots 와 같이 클러스터에 접근하는 사용자가 존재하며,
내부 요소간의 통신을 보안하고 인증 메커니즘을 통해 클러스터에 대한 관리 엑세스를 보안한다.

![[Pasted image 20240506144128.png|500]]
앞서 언급한 사용자중 End Users의 경우 애플리케이션 내에서 접근제한을 하기 때문에 제외하고, 
관리목적으로 사용자가 쿠버네티스클러스터에 엑세스 하는 것에 초점을 맞춰보자.

User, Service Accouts 두 유형의 사용자로 구분할 수 있다.
*쿠버네티스는 사용자 계정을 직접 관리하지 않는다. - 외부 소스에 의존*
세부정보나 인증서 있는 파일, LDAP같은 타사 ID 서비스가 이런 사용자들을 관리한다.

Service Account의 경우 쿠버네티스가 관리할 수 있다.
Service Account - not human, bots, process, services, applications 등

#### User
![[Pasted image 20240506144716.png|500]]
*모든 사용자 엑세스는 API 서버에 의해 관리된다.*
kubectl 명령이나 curl 을 요청하면 kube-apiserver로 요청이 가고, 이를 처리하기 전에 인증절차를 거친다.

#### Auth Mechanisms
어떻게 인증할 것인가.
- Static Password File
- Static Token File
- Certificates
- Identity Service (LDAP, Kerberos 같은 타사 인증 프로토콜)

 #### Basic - Static Password, Token File
![[Pasted image 20240506145206.png|600]]
csv 파일에 password, username, ID 목록을 저장하고 이 파일을
kube-apiserver 옵션으로 넘긴다.
그러기 위해서는 kube-apiserver.service 에 해당 옵션을 지정해야한다. 그리고 재시작.

![[Pasted image 20240506145223.png|600]]
kubeadm 도구를 사용해 클러스터를 설정했다면, yaml 파일에 옵션을 설정한 다음, 업데이트

![[Pasted image 20240506145234.png|500]]
다음과 같이.
기본자격증명을 통해 API 서버에 엑세스 할 수 있다.
Static Token File 도 이와 같다.

![[Pasted image 20240506145646.png]]
Static 파일에 사용자 이름, 암호,토큰을 저장하는 이 메커니즘은 불안정하기 때문에 추천하지 않는다.
쿠버네티스의 인증 절차를 이해하는 가장 쉬운 방식일 뿐이다.
또한 이를 kubeadm으로 할 경우 auth 파일을 통과할 볼륨마운트도 고려해야한다
-> Edit the kube-apiserver static pod configured by kubeadm to pass in the user details. The file is located at `/etc/kubernetes/manifests/kube-apiserver.yaml`

Create the necessary roles and role bindings for these users: - User에 대한 권한 부여 
Once created, you may authenticate into the kube-api server using the users credentials
`curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"`
https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/learn/lecture/14296208#content


## TLS Certificates(PRE-REQ)
### TLS Basic
#### SSL/TLS Certificates
Certificate - transaction 시 guarantee를 보장하기 위해 사용된다.
만약 사용자가 웹서버에 접근하고자 할때 TLS 인증서는 사용자와 서버사이의 통신이 암호화되도록 한다.
암호화가 없다면, 사용자가 서버에 보내는 정보를 쉽게 해킹하여 가져올 수 있게 된다.
따라서 전송하는 데이터를 암호화하여 보내야한다.
![[Pasted image 20240506150857.png|500]]
 *Symmetric encryption : 대칭암호화* - 데이터를 암호화하고 복호화하는 것을 같은 단일 키로
 암호화된 데이터를 서버가 받았을때 이를 decrypt 하기 위해 암호화 복호화할 수 있는 키를 같은 네트워크로 전송하여 서버는 해당 키를 통해 복호화하여 데이터를 볼 수 있다.
 -> 하지만 이때 대칭암호화를 하기 위한 키를 네트워크로 전송할때 해커가 이를 빼돌릴 수 있는데,
 이를 위해 비대칭 암호화를 사용한다.
 
*Asymmetric encryption : 비대칭 암호화* -  private key, public key=public lock
private key: 나만 갖고있을 수 있는 프라이빗한 키
public key: 누구나 접근 할 수 있는 공용 키
public key로 암호화했으면 private key로만 복호화 할 수 있다.

*Asymmetric encryption : 비대칭 암호화* 사용예시
![[Pasted image 20240506151637.png|500]]
*서버에 접근할 경우*
private key, public key 의 key pair을 생성한다음 public key를 서버에 등록한다. -> `cat ~/.ssh/authorized_key`
그리고 접속할 때 id_rsa 인 private key를 통해 인증하여 접속할 수 있다. 
`ssh -i id_rsa user1@server1`
다른 서버에 접속할경우도 마찬가지이다. public key를 복제하여 원하는 만큼 서버에 둘 수 있다.
 
다른 사용자가 접근 할 경우도 마찬가지다 key pair를 생성하여 public key를 서버에 등록한다.
ㅡㅡㅡ

다시돌아가 SSL/TLS 통신을 하여 데이터를 암호화하기 위한 대칭 암호화의 키를
비대칭 암호화를 사용하여 전달할 .수 있다.

서버에 private key, public key 의 key pair를 생성한다.
그리고 공개키를 client에게 전송하고, client는 대칭키를 전송받은 공개키를 통해 암호화 하여 서버에 전송한다.
서버는 이를 private key인 개인키로 복호화하여 대칭키를 얻는다.
![[Pasted image 20240506152337.png|600]]
해커가 아무리 중간에 빼돌려도 해커에게는 이를 복호화할 개인키가 존재하지 않기때문에 대칭키를 얻어낼 수 없다.

그다음, 서버와 클라이언트는 대칭키를 통해서 암호화된 데이터로 전송을 주고받을 수 있다. - *대칭암호화*
ㅡㅡㅡ

![[Pasted image 20240506153435.png|600]]
이때 만약 해커가 my-bank 라는 페이지를 똑같이 만들고 서버도 동일하게 만들어서 클라이언트의 네트워크를 조정하여 요청을 자신의 웹사이트로 오는것에 성공했다고 가정하자.
클라이언트는 https://my-bank.com 을 입력하여 들어갔을 뿐인데 해커의 웹사이트에 들어간것이고 가짜 서버에서 동일하게 key pair을 생성하여 공개키를 클라이언트에게 전송하고,
클라이언트는 대칭키를 공개키로 암호화하여 가짜서버에 전송한다. 이렇게 될경우 가짜서버가 대칭키를 얻게되고 클라이언트는 가짜 서버에게 전송하는 데이터들을 모두 해킹당하게 된다.
### CA
![[Pasted image 20240506153641.png|600]]
이를 방지하기 위해 서버가 정말 인증된 서버인지 확인하는 방법이 존재한다. -> *CA*
서버가 공개키를 전송할때, 키를 담고 있는 Cert 인증서를 함께 보낸다.
인증서를 살펴보면, 인증서가 누구에게 발급되는지에 대한 정보를 갖고 있다.
해당 인증서를 통해 서버가 진짜 서버인지 확인 할 수 있게 되는 것이다.

인증서는 아무나 만들수도 있다 따라서 이 인증서가 진짜인지 확인이 필요하다
인증서를 생성했으면 직접 서명이 들어가있어야한다. -> *sign*
해커에게 가짜 인증서를 받게되면 . 이 인증서가 가짜로 서명된 인증서임을 알 수 있게 되는데
이를 *브라우저*가 해준다.

![[Pasted image 20240506153836.png|600]]
브라우저에서 자물쇠 아이콘을 본적이 있을것이다. 이것이 바로 인증서가 진짜인지를 나타내는 것이다.

[How to create 인증서?]
Certificate Authority - CA : 인증서 유효성을 확인해주는 대표적인 기관
- CSR(Certificate Signing Request): 생성한 키(개인키)와 웹사이트의 도메인을 통해 인증서를 생성하여 CA에 요청을 보낸다. -> `openssl 명령어`
- Validate Information - 인증서 관계자가 상세정보를 보고 확인시 인증서에 sign
- Sign and Send Certificate

-> 해커가 보낸 가짜 인증서인 경우에 Validate Information 단계에서 실패하겠지
브라우저에는 브라우저가 신뢰하는 CA가 서명한 인증서가 존재한다.
[*CA자체가 합법적이라는 것은 브라우저가 어떻게 알것인가.*]
-> 브라우저가 이를 해준다.
 CA 마다 갖고있는 key pair가 존재하며 CA는 인증서에 서명할때 *개인키*를 사용한다.
 그리고 *공개키*들을 브라우저가 갖고 있기 때문에.
 브라우저는 공개키를 통해 복호화되는지를 확인하여 해당 인증서의 진위여부를 가릴 수 있게 된다.
**-> 하지만 개별적으로 호스팅된 사이트의 유효성을 검사하는 건 돕지않는다.**
이 경우 private CA를 호스팅할 수 있다. -> 회사 내에서 내부적으로 배포할 수 있는 CA 서버이다.
해당 개인 CA 서버의 공개키를 모든 직원의 브라우저에 등록하여 사용 할 수 있다.

![[Pasted image 20240506154158.png|600]]
따라서 정리해보자면,
[CA]
서버는 먼저 CA에 인증서 서명요청을 보낸다.
CA는 개인키로 CSR에 서명한다.
브라우저에는 CA의 공개키가 존재 - 모든 사용자는 CA의 공개키의 복사본을 갖고 있다.
서명된 키는 다시 서버로 보내진다.
서버는 서명된 인증서로 웹프로그램을 구성하고
[비대칭암호화]
클라이언트가 요청할때마다 서버는 먼저 생성한 key pair중에서 공개키로 인증서Cert를 보낸다.
사용자 즉, 클라이언트의 브라우저가 인증서를 읽고 서버의 진위여부를 체크한다. - CA의 공개키를 사용하여
진짜 서버임을 확인 했다면, 대칭키를 서버에게 받은 공개키로 암호화하여 전송한다.
서버는 개인키로 메시지를 복호화하여 대칭키를 회수한다.
[대칭암호화]
앞으로 해당 대칭키로 클라이언트와 통신이 가능해졌다. - 대칭암호화


앞서 클라이언트가 서버가 진짜임을 확인하는 과정을 살펴봤다면,
**서버는 클라이언트가 해커가 아닌 진짜인지 어떻게 확인 할 수 있을까?**
클라이언트에게 인증서를 요청

클라이언트는 한쌍의 키와 서명된 인증서를 생성해야한다. 유효한 CA로 부터.
그리고 서버로 인증서를 보내고 서버는 이를 통해 확인한다.
-> 단 이때, 웹사이트에 엑세스 하기 위해 인증서를 생성해본적이 없을 것이다.
그건 웹서버에선 TLS 클라이언트 인증서가 일반적으로 구현되어있지 않기 때문이다.
이는 호스트 아래에 모두 구현되어 있어, 일반 사용자는 인증서를 수동으로 생성하고 관리할 필요가 없다.
-> CA, Server, Cert 생성, 배포, 유지 프로세스를 포함한 . 이 전체적인 인프라를  *PKI* 로 알려져있다.

![[Pasted image 20240506154205.png|600]]
두 키는 짝을 이루는 키일뿐 암호화를 공개키로만 할수 있는건 아니다.
개인키로도 암호화 할 수 있다.
즉, 한쪽은 암호화, 한쪽으로 복호화할 수 있는 것이다. 같은 키로 복호화 암호화 둘다 할 수 없다.
공개키로 암호화했으면 복호화는 개인키 / 개인키로 암호화 했으면 복호화는 공개키로 해야한다.

HTTPS 원리 이해하기
https://brunch.co.kr/@growthminder/79

## TLS in Kubernetes

![[Pasted image 20240508002437.png|500]]
쿠버네티스는 마스터노드, 여러 워커노드의 집합이다.
이 노드들 간의 통신, 모든 서비스와 고객간의 통신 또한 쿠버네티스 클러스터 관리자와 kube-apiserver 간의 통신이나 kube-apiserver와 kube-scheduler 간의 통신.
모두 보안을 위해 TLS connection 이필요하다.
 -> 따라서 클러스터 내의 다양한 구성요소들에 대해
 서버인증서, 클라이언트 인증서를 사용해서 인증하며 통신해야한다.

![[Pasted image 20240508002617.png|600]]
위의 이미지와 같이 각각 구성요소는 모두 각각의 인증서crt와 private key가 필요하다. 
그리고 kube-api server 로 예시를 들면 이는 server로도 사용되지만 etcd server나 kubelet server와도 통신하기 때문에 client용의 crt와 key도 생성한다.


![[Pasted image 20240508002607.png|600]]
다음과 같이 필요한 인증서crt와 key는 매우많다.
client, server의 용도에 따라 구분할 수도 있다.
-> 그렇다면 이 인증서들을 생성하는 방법은?
앞서 설명한 것처럼 **"인증서 기관"**이 필요하다. -> Certificate Authority (CA)
모든 인증서에 서명하기 위해서는 말이다.

즉, 쿠버네티스는 클러스터에 적어도 하나이상의 인증서 인증을 요구한다.
CA는 한개 뿐 만 아니라 한개 이상으로 구성할 수도 있다.
![[Pasted image 20240508005012.png|500]]
예를들면 kube-apiserver 관련된 인증서 서명을 위한 CA, ETCD 를 위한 CA 로 여러개의 CA를 사용할 수 있다.

그리고, CA는 자체 인증서와 키가 존재한다. -> ca.crt, ca.key
### Generate Certificates

- EASYRSA
- OPENSSL
- CFSSL
![[Pasted image 20240508005702.png|600]]
1. openssl 명령어를 사용해서 private key를 생성.
2. 서명요청을 위해 방금 *만든 키와 함께 CSR* 생성 ( certificate CSR also contains your *public key*
	- CN: Common Name
3.  sign Cert -> ca.crt
	1. 이건 CA 자체에 대한 서명이기때문에 ca.key 인 첫단계에서 생성한 private key로 자체 서명
	2. 다른 인증서들에는 ca key pair를 사용하여 서명.
	
![[Pasted image 20240508005711.png|600]]
1,2번 과정은 위와 동일하지만
- sign cert 과정에서 ca.crt와 ca.key를 지정하여 서명. -> *CA key pair로 인증서에 서명*
	따라서 클러스터내의 유효한 인증서가 된다. -> admin.crt 가 생성.
	관리자가 쿠버네티스 클러스터에 인증할때 사용하는 인증서가 된다.
	마치 인증서는 USERID, key는 Password와 비슷하다.
- `Group: SYSTEM:MASTERS` 로 그룹을 지정해준다. -> /OU=system:master 에 지정.

![[Pasted image 20240508002748.png|600]]
kube-scheduler는 시스템 컴포넌트로 controlplane의 pod다. -> name에 prefix로 `SYSTEM` 이 붙어야한다.
kube-controller manager, proxy 등도 마찬가지이다.

![[Pasted image 20240508002806.png|600]]
관리자가 kube-apiserver에 REST API 통신하고자 할때 
1. 앞서 만든 admin 키와 인증서 그리고 CA 인증서를 사용해서 통신할 수 있다.
2. 또는 kube-config.yaml 에 속성으로 등록해준다. -> 대부분 이방식 사용

![[Pasted image 20240508010820.png|600]]
앞서 TLS 통신에서 보낸 인증서의 유효성 검사를 하기 위해서 copy public certificate 이 필요하다고 했다.
웹어플리케이션의 경우 브라우저에 이미 설치되어있지만,
쿠버네티스 환경은 아니기 때문에 *CA Root 인증서*를 모두 복사해 넣는다.

![[Pasted image 20240508002821.png|600]]
인증서 생성 절차는 위와 동일하다.
etcd는 고가용성을 위해 *다중 서버*에 걸처 생성되기 때문에
*peer*가 존재하고 이에 따라 인증서와 키도 더 생성할뿐만 아니라 설정 etcd.yaml 파일을 보면 peer-cert-file을 지정했음을 볼 수 있다
그리고 앞서 언급한 내용대로 Root CA 의 path도 존재한다.

![[Pasted image 20240508002846.png|600]]
kube-apiserver는 클러스터 리소스중 가장 많은 통신을 한다. -> Kube API 서버가 마치 쿠버네티스와 같기때문에
*kubernetes.default* 라고도 하며 아래의 이름들로 불린다.
	- kubernetes
	- kubernetes.default
	- kubernetes.default.svc
	-  kubernetes.default.svc.cluster.local
	- ip address (kube api 서버를 실행하는 호스트의 ip 주소)
	*해당 이름들은 모두 kube-apiserver 용 인증서에 존재해야한다.*
how to 해당 이름들을 명시할까?
`-config openssl.cnf` 
openssl.cnf 에 설정하여 CSR 생성시 config에 옵션으로 넣어준다.

[kubelet]
![[Pasted image 20240508002910.png|600]]
kubelet은 노드마다 존재하기 때문에 노드마다 인증서가 존재한다.
api 서버가 노드와 통신하기 위해 Kubelet 과 통신하고 이때 인증서를 사용하게 될 것이다.
kubelet-config 파일에 설정하여 사용할 수 있다.

증명서 이름: 어떤 노드가 인증을 하는지 알아야하기때문에
`system:node:(node name)`
`Group: SYSTEM:NODES`

### View Certificate Details

![[Pasted image 20240508012028.png|600]]
대부분 kubeadm 방식으로 cert 설정값 확인.

![[Pasted image 20240508011947.png|500]]
인증서 내용 확인
 - issuer 이름
 - 유효일자.
 - Common Name
 - Alternative Name
#### Log 확인
![[Pasted image 20240508012016.png|500]]

![[Pasted image 20240508012001.png|500]]

![[Pasted image 20240508012322.png|500]]
#### test

kube-api server crt
![[Pasted image 20240508000459.png|600]]
![[Pasted image 20240508000609.png|600]]


Identify the ETCD Server Certificate used to host ETCD server.
![[Pasted image 20240508000959.png|600]]

What is the Common Name (CN) configured on the Kube API Server Certificate?
**OpenSSL Syntax:** `openssl x509 -in file-path.crt -text -noout`
![[Pasted image 20240508001158.png|600]]

트러블 슈팅 문제
1. k get po 안되는 오류
`docker ps -a | grep `
`docker logs (container ID)`
을 통해 로그를 확인한 후 
로그에 표시된 위치에 해당 인증서가 없음을 확인

`vi /etc/kubernetes/manifests/kube-apiserver.yaml`
인증서 path 수정 후 해결

2. k get node 안되는 오류
위와 동일한 방법으로 트러블 슈팅
`docker ps -a | grep kube-apiserver`
`docker ps -a | grep etcd`
`cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep "\-\-etcd"

### Certificate API

CA Server는 매우 안전하게 유지되어야 하는데
kubernetes 에서는 Master node 에 위치해 있다.

사용자가 증가하고 인증서 서명요청이 증가함에 따라
인증서 서명요청이 자동화된 방법이 필요하다. 인증서가 만료되면 Resign도 되도록
![[Pasted image 20240508012937.png|500]]
kubernetes에는 built-in인 Certificate API가 존재하여 이를 대신 해줄 수 있다.
인증서명요청을 해당 API를 통해 직접 요청보낸다.
위 사진과 같이 object를 생성하고 review 요청을 보내며, 이를 approve하면 cert를 사용자에게 공유해준다.

![[Pasted image 20240508013125.png|500]]
사용자가 key를 생성하고
해당 키를 통해
openssl req 요청(csr)을 보내면
관리자는 CSR Object를 생성하는데 manifest file(yaml)으로 생성하며
certificate 부분은 base64 인코딩하여 추가한다.
개체가 생성되면 `kubectl get csr` 로 조회할 수 있으며

![[Pasted image 20240508013309.png|500]]
new request를 확인하고 approve를 한다.
![[Pasted image 20240508013730.png|500]]


![[Pasted image 20240508013836.png|400]]
Controller Manager : 인증서 관련 작업 실행
- CSR-Approving
- CSR-Signing

![[Pasted image 20240508013955.png|500]]
따라서 인증서에 서명하기 위해서는 Root CRT 인증서와, key가 필요하기 때문에 구성요소를 보면 포함되어있는 것을 확인 할 수 있다.

#### test
CSR Object 생성후
approve
deny