## Readiness Probe & Liveness Probe

## **Readiness Probe**
pod 에는 lifecycle이 있다. 
처음 create 되면 pending -> scheduler에 의해 node에 배치된다면 containerCreating으로 컨테이너 create하고 있는 상태가된다. (만약 배치가 안되면 pending상태에 머문다) -> 모든 컨테이너가 정상적으로 create된다면 Running상태가 된다. 
하지말 실질적으로 생각하면 모든 컨테이너가 create돼도 실제 사용자가 해당 어플리케이션을 사용하고자한다면 어플리케이션 서버가 뜨는시간 nginx, ui 뜨는시간 등 시간이 더 소요된다. 
이런경우 사용자가 실질적으로 사용할 수 있는 상태인지 어떻게 테스트하고 알 수 있는가
이때 사용하는게 바로 *Readiness Probes*이다. 
![[../../resources/Pasted image 20230606220020.png|400]]
![[../../resources/Pasted image 20230606220034.png|500]]
위의 정의한 api에 응답이 오면 사용할 수 있는 상태인것을 알 수 있게 된다.

![[../../resources/Pasted image 20230606220137.png]]

periodSeconds: specify how often to probe
failureThreshold: 몇번까지 시도 가능한지 정의 

## **Liveness Probes**
만약 pod에 있는 어플리케이션이 어느 버그로 인해 무한루프를 돌고있다 하자. 그럼 이를 어떻게 알고 파드를 재시작할것인가
이때 사용하는게 바로 *Liveness Probe*이다. 
A liveness probe can be configured on the container to periodically test whether the application within the container is actually healthy. If the test fails, the container is considered unhealthy and is destroyed and recreated.

![[../../resources/Pasted image 20230606220637.png|500]]

## **Logging**
![[../../resources/Pasted image 20230606223755.png|500]]
docker logging 방법
-d: 백그라운드에서 실행으로 터미널에 로그를 보여주지 않게 할 수 있다. 
따로 log 보기 위해선 
`docker logs -f (container id)`
-f : live log trail

![[../../resources/Pasted image 20230606223929.png|500]]

![[../../resources/Pasted image 20230606223950.png|500]]
멀티컨테이너면 뒤에 컨테이너를 지정해줘야한다. 

## **Monitoring**

### *Metrics Server*
쿠버네티스 클러스터당 1개
pod에서 메트릭 회수 - 메트릭 모아서 메모리에 저장
-> 메트릭 서버는 *IN-MEMORY* 모니터링 솔루션이다. 
메트릭을 disk에 저장하지 않는다. 
그래서 historical 한 data는 볼 수 없다. 

어떻게 메트릭을 모으는가 
*kubelet*을 실행한다. 
kubelet- 쿠버네티스 api master server로부터 지시를 받고 포드를 실행시키는 역할
cAdvisor의 하위요소를 포함하고 있다. 
cAdvisor는 포드에서 성능 메트릭을 회수 하고 kubelet api통해 메트릭을 공개해 메트릭 서버에서 메트릭을 사용할 수 있도록 한다. 

in minikube
`minikube addons enable metrics-server`
하거나 
github에서 clone으로 다운후 실행

metric server가 메트릭 모아 저장하기까지 기다린다음 
 
`kubectl top node`
각 노드의 cpu 메모리정보 제공 
`kubectl top pod`
각 파드의 cpu 메모리정보 제공 

