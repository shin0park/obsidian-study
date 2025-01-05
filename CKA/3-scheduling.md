
## Manual Acheduling
![[Pasted image 20240324222518.png|600]]
pod yaml을 보면 nodeName이란 속성을 볼 수 있는데,
이는 기본값이 아니며 쿠버네티스가 스케쥴러에 의해 자동으로 채워주는 값이다. 
스케줄러는 nodeName의 값이 없는 파드들을 식별하여 스케줄러 알고리즘을 통해 pod를 적절한 node에 bind 한다.

노드를 모니터링하고 스케줄링할 스케쥴러가 없다면, how??
직접 포드를 노드에 할당 할 수도 있다.
-> pod를 생성할때 yaml 속성의 nodeName에 노드를 지정하는 것이 가장 간단하다.
단, pod의 nodeName 속성에 노드를 지정하는 것은 파드 생성지점에만 지정할 수 있다. 

이미 파드가 생성된 경우라면??
![[Pasted image 20240324223042.png|600]]
`kind: Binding`
바인딩객체를 생성해야한다.
이때 실제 스케줄러가 하는 일을 모방하여, 바인딩 객체에 파드를 생성할 노드를 지정하고 pod 바인딩 API에 POST 요청을 보낸다.


## Labels and Selectors
![[Pasted image 20240324224345.png|400]]
다음 이미지와 같이 각기 다양한 오브젝트들이 수없이 많이 존재하게 될것이다.
![[Pasted image 20240324224411.png|400]]
이를 공통된 것끼리 묶고, 식별해내야 관리하기 유리할 것이다. 
이때 사용하는것이 Labels and Selectors 이다.

![[Pasted image 20240324224445.png|600]]

`kubectl get po --selector app=App1`


![[Pasted image 20240324224617.png|600]]
대표적으로 replicaset을 보면, 
파드에 Label을 설정하고 replicaset의 selector를 통해 파드를 그룹으로 묶는다.
**위의 metadata는 replicaset의 label이며, 아래의 template의 label이 파드의 label이다.

replicaset을 찾아 개체를 구성할때는 replicaset의 라벨을 사용할 것이고, 
pod들을 replicaset으로 묶을때는 replicaset의 selector와 pod의 label을 가지고 하나의 그룹으로 묶는 것이다.

**따라서 replicaset의 seletor의 matchLabels는 pod의 라벨 중에 일부여야만 한다.

![[Pasted image 20240324225209.png|600]]
이는 service에서도 마찬가지로 적용된다. service의 seletor를 통해 해당 서비스가 어느 파드에 적용될것인지 구별할 수 있다.

## Annotations
![[Pasted image 20240324225344.png|600]]
앞선 label 과 selector은 그룹과 개체를 매치하는데 사용됐다면, annotation은 다른 세부사항들을 기록하는데 사용된다.

### test

```
 k get po --show-labels
 ```
 ![[Pasted image 20240324225845.png]]
![[Pasted image 20240324230119.png]]
![[Pasted image 20240324230200.png]]


## Taints and Tolerations

해당 강의에서는 사람은 노드고 벌레는 포드에 비유하며 설명한다.
-> Taints and Tolerations는 하나의 노드에 어떤 포드들로 스케쥴링할 수 있는지 제한을 설정하기 위해 사용된다.

![[Pasted image 20240324230702.png|400]]
지금처럼 어떠한 제한도 없을때는 모든 노드에 포드를 배치해 균형이 맞도록 한다.
![[Pasted image 20240324230746.png|400]]

만약 node1에 D만 배치하게 하고 싶다면,
![[Pasted image 20240324230922.png|400]]
node1에는 taint=blue를 설정하고 이를 D만 수용하게 하기 위해 D에 blue에 대한 toleration을 설정한다.

**단, D가 Node1에 지정될 수 있는 거지 항상 node1으로 간다는 보장은 이것만으로는 없는것이다. 다른 노드들도 갈 수 있기 때문이다. 만약 하나의 노드에 파드가 지정되도록해야한다면, node affinity라는 개념을 사용해야한다.

![[Pasted image 20240324231434.png]]
NoSchedule: 해당노드에 스케쥴 안되는 것
prefernoSchedule: 스케쥴안되는것을 선호하지만 장담할 수 는 없는
NoExecute: 새 포드가 노드에 지정되면 안되고, 만약 조건을 어기는 파드가 노드에 있다면 퇴거해야하는 것
### Tolerations
![[Pasted image 20240324231533.png|300]]
pod의 정의파일에서 tolerations 속성에 key=value를 지정해줘야 한다.

![[Pasted image 20240324232039.png|600]]
**마스터노드에는 어떤 포드도 스케줄링하지 않는데, 그이유를 이를 통해 알 수 있다.
-> 쿠버네티스 클러스터가 처음 설정되면 마스터 노드에 taint 설정이 자동으로 되기 때문이다.

### test
- Delete taint
![[Pasted image 20240324233131.png]]

## Node Selectors
![[Pasted image 20240324233354.png|400]]
위의 이미지처럼 리소스가 더 많은 node1에 더 작은 파드가 배치되는 경우들이 있을 것이다. 
이때 특정 파드를 특정노드에만 배치하고 싶다면??
simple and easy한 방법이 바로 Node Selector이다.
![[Pasted image 20240324233535.png|400]]
nodeSelector에 key:value로 원하는 노드에 설정된 label을 등록해주면된다. 
그럴려면 pod 생성전에 노드에 label을 붙여줘야 될것이다 
![[Pasted image 20240324233702.png|500]]

이때 만약, 
- Large or Medium 
- not Small
의 더 복잡성이 있는 배치가 필요하다면 **Node Affinity**를 사용해야한다.

## Node Affinity
목적: pod가 특정 node에 host 될 수 있도록 하는 것이다.
![[Pasted image 20240324234008.png|600]]
위에서 node seletor로 설정한 특정노드에 파드 호스팅과 동일하게 node affinity로 설정한 것이다.
좀더 디테일하고 복잡한 조건들을 위의 문법과 같이 설정할 수 있다.
operator : in, notin, exists ...

![[Pasted image 20240324234316.png|600]]
![[Pasted image 20240324234648.png|600]]

required: 조건에 알맞는것이 없으면 스케쥴링 할 수 없다. (일치하는 노드가 없으면 pod는 배치되지 않는다.)
prefer: 일치하는 노드가 없어도 파드 배치는 발생한다. 일치하는것이 없으면 규칙을 무시하고 모든 가능한 노드에 파드를 배치한다. (파드 배치보다 파드 자체 실행이 더 중요한 경우)
ignored: any change

### test
![[Pasted image 20240324235148.png|600]]




pod들을 각각 알맞는 색의 노드들에 배치하게 하고싶다면,
먼저 taints and tolerataions를 사용하여 
Pod에 각각 Tolertaions를 설정하여 색상을 부여하고, node들에는 특정 색상만 배치할 수 있도록 taint를 설정한다.
![[Pasted image 20240324235934.png|400]]

![[Pasted image 20240324235948.png|400]]
하지만 taint를 한들 red pod가 Red node로 갈수 있는 것일뿐 other node에 배치될 수 도 있다. 
이때 사용하기 좋은 것이 Node Affinity이다.

![[Pasted image 20240325000203.png|400]]
각 노드에 라벨을 붙이고, 각 pod에 node affinity를 설정하면, red가 다른 노드로 간다는 문제는 없어진다.
이때 이전에 설정한 것처럼 taint and tolerations 을 설정안한다면 일반 node가 red로도 갈 수 있기때문에 두가지 모두 설정해야 요구사항에 부합하도록 파드들을 배치할 수 있게 된다.


