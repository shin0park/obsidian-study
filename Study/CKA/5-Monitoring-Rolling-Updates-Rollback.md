쿠버네티스 자원 모니터링

어떤 자원들을 모니터링 할까?
노드레벨의 메트릭
노드의 수 healthy 측정, CPU, 메모리 
파드레벨의 메트릭
파드의 수, 파드의 성능, CPU, 메모리

![[Pasted image 20240407192153.png|400]]
쿠버네티스 자체에는 이러한 리소스를 모니터링하는 탑재된 모니터링 솔루션이 없지만,
-> 사용가능한 오픈소스 솔루션은 많이 존재한다.
Metric Server, 프로메테우스, Datadog, Elastic Stack, dynatrace

### Metrics Server
**Heapster** - 쿠버네티스 초기에 모니터링과 분석을 가능하게 한 프로젝트였는데, 현재는 deprecated 됐으며,
간소화된 버전인 Metric Server가 만들어졌다.

![[Pasted image 20240407192307.png|600]]
쿠버네티스 클러스터당 Metric server는 1개이다. 
메트릭서버는 파드에서 메트릭을 수집하여 메모리에 수집한다. - **In-Memory** 모니터링 솔루션이다.
디스크에 메트릭을 저장하지 않느다.
디스크에 저장하기 위해서는 앞서 다룬 다른 모니터링 시스템을 사용해야한다.

Kubelet은 cAdvisor라는 리소스를 갖고 있는데,
cAdvisor는 파드에서 메트릭을 회수하고 kubelet API를 통해 메트릭 서버에서 메트릭을 볼 수 있도록 한다.

![[Pasted image 20240407193013.png|600]]
로컬클러스터로는 minikube를 사용하면
addons로 metric-server를 활성화 할 수 있다.
다른 모든 환경에서는 git에서 clone 후 apply하여 배포하여 사용할 수 있다.
![[Pasted image 20240407193213.png|600]]
다음과 같은 명령어로 노드와 파드의 사용량을 확인 할 수 있다.

### Application Logs

![[Pasted image 20240407193907.png|400]]
event-simulator: 랜덤 event 웹 서버 시뮬레이션
 ![[Pasted image 20240407193919.png|400]]
 -d : background 옵션 가능
 `docker logs -f (container id)`
로 로그확인 가능.

![[Pasted image 20240407211522.png|600]]
kubernetes 환경에서도 동일하게 pod로 생성하여 log 확인 가능하다

`kubectl log -f (pod name) (container name)`
container 여러개인경우 container name을 명시해야한다.

## Application Lifecycle Management
### Rolling Updates and Rollbacks

#### Rollout and Versioning
![[Pasted image 20240407212445.png|500]]
배포를 하면, Rollout이 트리거 되는데,
new Rollout은 new deployment Revision (Revision1)을 생성한다.
이후 만약 애플리케이션이 업그레이드 되어 컨테이너 버전이 업데이트된다면,
new Rollout이 트리거되고, new deployment Revision(Revision2)가 생성된다.
 ->이는 배포를 추적하고 필요시 이전 버전으로 rollback 할 수 있게 한다. 

![[Pasted image 20240407212925.png|400]]
rollout 명령어를 deployment 이름과 함께 사용하여 
배포한 status와 history를 확인 할 수 있다.

![[Pasted image 20240407212459.png]]
두가지의 배포 방식이 존재한다.
Recreate: 위의 이미지와 같이 복제본이 5개로 할때, 모두 destory하고 모두 create
-> 문제: 전체가 다운되고 다시 create되는 동안 application을 사용 할 수 없다 - 순단이 생긴다.
Rolling Update: one by one 으로 구버전 내리고 새버전을 올리는 방식 -> default strategy

![[Pasted image 20240407212511.png|600]]
yaml파일의 image 부분을 수정후 apply 하거나
set image 명령어를 사용하여 container image을 업그레이드 할 수 있다 - 단 set image 명령어를 사용하면 정의파일과 달라진다는 점에서 유의해야한다.

![[Pasted image 20240407212551.png]]
recreate은 0이 됐다가 5로 증가함을 확인 할 수 있는데
rolling update는 한개씩 축소 생성됨을 확인 할 수 있다.

![[Pasted image 20240407212533.png]]
새 버전으로 업그레이드 하게 되면
새로운 replica set이 생성되고 기존 replicaset의 파드가 한개씩 삭제될때마다 파드가 한개씩 create된다.

![[Pasted image 20240407212540.png]]
rollout undo 로 rollback 한다면, 이전 replicaset에 다시 파드가 5개가 생긴 것을 볼 수 있다.

![[Pasted image 20240407214022.png]]

