
## Application command
![[Pasted image 20240414190657.png]]
ubuntu 이미지 인스턴스를 실행하고 종료하게된다.
docker ps로 현재 실행중인 컨테이너를 조회할때는 조회되지 않지만 -a옵션으로 exited 된 컨테이너까지 조회하도록 했을때 조회됨을 볼 수 있다. 

why?

virtual machine과는 달리 컨테이너는 운영체제를 호스팅 하지 않는다. 컨테이너는 특정 작업이나 프로세스를 실행한다. 컨테이너는 웹서버, 애플리케이션 서버 데이터베이스 및 기타 연산 분석 작업들에 사용된다. 그리고 **해당 작업이 완료되면 컨테이너는 exited** 된다. 

그럼 컨테이너 내에서 실행되는 프로세스는 누가 정할까?

![[Pasted image 20240414191202.png]]
docker 파일을 예시를 들면
CMD가 존재하는데, 이를 통해 실행될 프로그램을 정의한다.

![[Pasted image 20240414191429.png|400]]
이전의 `docker run ubuntu` 를 실행한 이미지에 대한 docker 파일을 보면
default cmd도 bash를 실행함을 알 수 있다.
기본적으로 docker는 실행중일때 컨테이너에 terminal 연결하지 않기때문에 
실행된 bash 프로그램이 terminal을 찾지 못해 exited 됐던 것이다.

![[Pasted image 20240414195116.png|600]]
다음과 같이 5초 sleep 하도록 동작하게 하기 위해 CMD를 추가하는데 
**이때 명령어와 파라미터는 각각 다른 element로 구성되어야 한다. 즉 배열의 각각 요소로 기입되어야 한다.**

![[Pasted image 20240414195556.png|600]]
여기서 sleep의 초를 입력받은 값으로 지정하고 싶다면,
**ENTRYPOINT** 진입점 을 사용 할 수 있다. **Entrypoint는 시작시 실행되는 명령어이다.**
docker run 명령시,
Entrypoint와 run 명령어와 함께 입력한 값인(10)이 합쳐져 `sleep 10`명령이 수행된다. 
그런데 진입점을 설정했는데, run 명령어 실행시 해당 인자값을 입력하지 않으면, 위의 이미지와 같이 에러가 발생할 것이다.
이때 CMD 명령어에 위의 이미지와 같이 "5"를 기입하여 기본값을 설정하면, default값으로 5가 입력되어 에러가 발생하지 않게 된다. 또한, run 명령어와 함께 10이란 인자를 입력하면 10으로 덮어져 `sleep 10`이 실행된다.

위에서 살펴본 docker 예시를 
kubernet pod로 정의를 할 수 있다. 

*같은 dockerfile로 만들어진 image를 사용하는 pod라고 하면 pod 에 정의된 cmd, arg로 덮어씌워질 수 있다.* -> *override*

pod 정의파일을 보면 containers 하단의 command와 args를 정의할 수 있으며, 
이는 docker파일의 
*ENTRYPOINT = command
CMD = args*
와 동일하다.
```
pods/commands.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: command-demo
  labels:
    purpose: demonstrate-command
spec:
  containers:
  - name: command-demo-container
    image: debian
    command: ["sleep2.0"]
    args: ["10"]
```

---
```
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
    - "sleep"
    - "5000"
```

```
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "5000"]
```

```
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep"]
    args: ["5000"]
```
위의 세가지 정의 내용은 동일하다.

*참고. 
- *command는 항상 배열이다*
- *배열에서 "" 를 입력해줘야한다.*
```
     command:
    - "sleep"
    - 5000(x) "5000"(o)
  ```
- *명령을 지정할때 명령은 항상 첫번째 항목에 있어야한다*
	sleep = command
	5000 = argument

만약 여기서 5000을 2000으로 바꾸고자 할때,
여러 가지 방법이 존재한다. 
1. edit 
   edit을 할 경우에 바꿀수 없는 속성으로 에러가 발생되고 수정한 내용은 /tmp/new.yaml 형태로 저장된다.
   그렇다면 기존 리소스 지우고 새로 생성해야 하는데, replace 명령어로 빠르게 할 수 변경할 수 있다.
   `k replace --force -f /tmp/new.yaml`
2. k get po -o yaml > new.yaml
	으로 새로운 정의파일을 생성항 다음, 기존 리소스 지우고 새로 생성 - 하지만 위와 마찬가지로 replace 명령어로 빠르게 변경할 수 있다.
	`k replace --force -f new.yaml`

애초에 kubectl run으로 pod 생성시 cmd argument를 지정해줄 수도 있다. kubectl run --help 참고.
```
  # Start the nginx pod using the default command, but use custom arguments
(arg1 .. argN) for that command
  kubectl run nginx --image=nginx -- <arg1> <arg2> ... <argN>
  
  # Start the nginx pod using a different command and custom arguments
  kubectl run nginx --image=nginx --command -- <cmd> <arg1> ... <argN>
```

`kubectl run webapp-green --image=kodekloud/webapp-color -- --color green`
`kubectl run webapp-green --image=kodekloud/webapp-color --command -- python app.py -- --color green`

## Environment Variables

쿠버네티스의 환경변수 설정. 
![[Pasted image 20240414200448.png|500]]
docker에서는 
`docker run -e APP_COLOR=pink simple-webapp-color`

kuber의 pod 정의에서는 
```
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp-color
spec: 
  containers:
  - name: simple-webapp-color
    image: simple-webapp-color
    ports:
      - containerPort: 8080
    env:
      - name: APP_COLOR
      - value: pink
```
로 정의한다.
*env는 배열이다*

위의 방식이외에 다른방식도 존재하는데 
즉 환경변수를 정의할때 3가지 방법이 존재한다. 

1. plain key value
2. ConfigMap
3. Secrets

![[Pasted image 20240414200502.png|400]]
값을 지정하는 대신 configMap이나 secret의 사양에 따르게 할 수 있다. 

1번으로 할 경우 env가 많아지면 관리하기가 여려워지게 되며,
2,3번으로 따로 분리하여 정의 후 사용한다면 env을 중앙에서 쉽게 관리 할 수 있게 된다.

### ConfigMap
configMap으로 env 변수들을 중앙관리히여 보다 더 쉽게 관리할 수 있게 한다.

![[../../resources/Pasted image 20230529185813.png|500]]
#### create configMap
`kubectl create configmap`
-> imperative 하게 생성할 수도 
`kubectl create -f` 
-> declarative 하게 생성할수도 있다.

### imperative
```
kubectl create configmap \
	<config-name> --from-literal=<key>=<value>

 kubectl create configmap \
	app-config --from-literal=APP_COLOR=PINK

 kubectl create configmap \
	app-config --from-literal=APP_COLOR=PINK \
			   --from-literal=APP_MOD=prod \
```
--from-literal 옵션은 명령 자체에서 key, value 지정하는데 사용된다.

#### from file
```
kubectl create configmap \
	<config-name> --from-literal=<path-to-file>

kubectl create configmap \
	<config-name> --from-literal=app_config.properties

```
### declarative
![[../../resources/Pasted image 20230529190647.png|500]]
`kubectl get configmaps`
`kubectl describe configmaps`
으로 확인가능 

### inject into Pod

![[../../resources/Pasted image 20230529191015.png|400]]
*envFrom  배열이다*

### Secrets
secrets도 configmap 처럼 imperative, declarative한 방법으로 나뉜다. 
`kubectl create secret generic`

![[../../resources/Pasted image 20230529220827.png|600]]
![[../../resources/Pasted image 20230529220851.png|600]]
declartive하게 생성할 경우 인코딩된 형식의 값을 data value값으로 지정해야한다.

*기본적으로 secret은 base64 암호화한다.* 
따라서 누구나 base64로 decode할 수 있기 때문에 암호화 되어있다고 할 수 없긴하다. 
하지만 평문으로 저장하는것보단 안전하다.
`echo -n "mysql" | base64`
`echo -n "6=4sdf" | base64 --decode`

```
k get secrets
k describe secrets
k get secret app-secret -o yaml
```

![[../../resources/Pasted image 20230529221129.png|500]]

![[../../resources/Pasted image 20230529221154.png|500]]

#### Note on Secrets
- Secrets are not encrypted. Onlt encoded.
	- github 등의 외부로 푸시할 경우 secrets 정의 파일을 코드와 함께 check in 하지 않아야 한다.
- Secrets are not encrypted in ETCD.
	- Enable encryption at rest 
	- -> ETCD에 암호화되어 저장되도록 *encryption at rest*를 활성화하는 것을 고려해라.
	  https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
- Anyone able to create pods/deployments in the same namespace can access the secrets.
	- Configure least-privilege access to Secrets - RBAC
- Consider third-party secrets store providers AWS Provider, Azure Provider, etc..
	- AWS - secretsmanager

https://kubernetes.io/docs/concepts/configuration/secret/#protections
*kubernetes handles secrets*
secret은 해당 node의 pod에 필요한 경우에만 node로 전송된다.
kubelet은 secret이 disk storage에 write 되지 않도록 tmpfs에 secret을 저장한다. 
그리고 secrets를 사용하는 pod가 삭제될때마다 kubelet은 그 secrets의 local copy data도 같이 삭제한다.

*encryption at rest* 실습
https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
`apt-get install etcd-client`
`etcdctl`
![[Pasted image 20240414213239.png|500]]
-> 데이터는 암호화되지 않은 포맷으로 저장됨을 알 수 있다.

`ps -aux | grep kube-api | grep "encryption-provider-config"`
으로 *encryption at rest*  가 이미 활성화 되어있는지 체크한다.


https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#understanding-the-encryption-at-rest-configuration
![[Pasted image 20240414214128.png|500]]
암호화 알고리즘을 선택해서 keys를 입력하여 정의파일을 생성한다.
EncrpytionConfiguration 정의파일을 /etc/kubernetes/enc 위치로 이동한다.(추후 kube-apiserver의 volume으로 지정할 위치)

https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/#use-the-new-encryption-configuration-file
![[Pasted image 20240414214400.png|500]]
kube-apiserver pod 정의파일에 -> `vi /etc/kubernetes/manifests/kube-apiserver.yaml`
- add command - encryption-provider-config=/etc/kubernetes/enc/enc.yaml
- volumeMount, volumes 설정

*참고. 위의 과정으로 encryption at rest 을 설정하기 이전의 secrets은 etcd에 암호화되어 저장되지 않는다. 설정 후에 생성한 secrets들만 해당된다.*
so, 모든 데이터를 같은 데이터로 업데이트 하여 모두 설정되도록 할 수 있다.
`kubectl get secrets --all-namespaces -o json | kubectl replace -f -`


## Multi Container Pod
pod 정의 yaml 파일에 spec아래에 containers가 배열이다. 여기에 multi로 container 정의 할 수 있다.

*CKAD*
### Design Patterns
3가지 패턴이 존재한다. 
![[../../resources/Pasted image 20230604170220.png|400]]

1. SIDECAR
   ex) 각각 파드에 이 로그를 수집하는 log 수집용 컨테이너를 배포한다. 
   해당 컨테이너가 로그를 중앙 로그서버에 전송하고 중앙로그서버에 로그를 모은다. 
 2. ADAPTER
    중앙로그서버에 로그를 전송할때 파드마다 로그의 format이 각기 다를것이다. 
    이를 하나의 format으로 통합해주는 adapter 컨테이너를 함께 배포한다. 중앙로그서버에 전송전 adapter가 format을 통일시켜준다. 
 3. AMBASSADOR
    dev test prod 환경에 따라 연결해야될 db가 다를텐데 이때 ambassador 컨테이너는 proxy 역할로 각 맞는 db를 연결해준다. 
    
    
![[../../resources/Pasted image 20230604170433.png|400]]
![[../../resources/Pasted image 20230604170501.png|400]]

### **log 확인**
```
kubectl logs (pod name)

ex)
kubectl logs app -n elastic-stack
kubectl exec -it app -n elastic-stack -- cat /log/app/log
```