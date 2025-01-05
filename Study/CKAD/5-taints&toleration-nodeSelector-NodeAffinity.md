## **Taints and Toleration**
![[../../resources/Pasted image 20230603233040.png]]
벌레에 비유하자면
toleration: 벌레- pod
taint: 스프레이- node
이다. 
*taint는 node에 적용하며 toleration은 pod에 적용한다.*

node1에 D라는 파드만 배치시키고 싶다고 할때 tains and toleration을 적용한다 보면된다. 
![[../../resources/Pasted image 20230603233216.png|500]]
taint는 
`kubectl taint nodes node-name key=value:taint-effect`
다음과 같은 명령어를 통해 생성한다. 
위의 사진과 같이 taint-effect는 *3가지*가 존재한다. 

![[../../resources/Pasted image 20230603233324.png|500]]
toleration과 같은 경우 pod의 정의 spec아래에 정의해주면된다. 



![[../../resources/Pasted image 20230603233411.png|500]]
생각해보면 master노드도 있는데 파드가 worker node에만 배치되는걸 알 수 있다. 
이는 처음 쿠버네티스 클러스터가 실행될때 master 노드에 taint가 설정된 것이다. 
위의 사진의 명령어대로 des를 해보면 noSchedule임을 알 수 있다. 

#### **pod가 어느 노드에 떠있는지 알고자 할때**
`kubectl get po -o wide`

## **Node Selectors**

만약 pod가 세개 있다고 할때 많은 리소스를 사용할 pod가 존재한다면 node의 자원이 큰곳에 배치하는게 좋을 것이다. 이럴때 node selector를 사용한다. 

![[../../resources/Pasted image 20230603235625.png]]
replica set 등의 개념에서 label로 구분을 했던 방식처럼 
pod의 정의에 spec 아래에 nodeSeletors 에 key: value를 정의한다. 

이 작업전에 node에 이미 key:value label이 정의되어있어야한다. 즉 해당 라벨과 일치하는 node에 배치하고자 한다는 의미이다. 

![[../../resources/Pasted image 20230603235850.png|500]]
위의 사진의 명령어로 node에 label을 정의한다. 

하지만 예를들어 large or medium인 node에 배치하고 싶다던가 not small인 노드에는 모두 배치해도 된다는 등의 복잡한 요구사항을 node seletor로 하기는 힘들것이다. 

## **Node Affinity**
node 선호도 
![[../../resources/Pasted image 20230604001254.png]]

![[../../resources/Pasted image 20230604001302.png]]
node affinity를 사용하면 다음과 같이 더 디테일하게 node를 지정 할 수 있다. 

![[../../resources/Pasted image 20230604001353.png|444]]

![[../../resources/Pasted image 20230604001338.png|400]]

만약 pod 배치후 배치된 node의 라벨이 삭제된 경우엔 어떻게 될까?
또한 affinity를 정의했는데 해당하는 node가 한개도 없을 땐 어떻게 될까. 
여기엔 3가지 type이 존재한다. 

requiredDuringSchedulingIgnoredDuringExecution
은 required로 해당하는 node가 없으면 pod를 배치하지 않는다. 
중간에 node의 라벨을 삭제해도 pod의 위치는 변함없으며 상관없이 실행된다. 

preferredDuringSchedulingIgnoredDuringExecution
은 preferred로 해당하는 node가 없으면 아무 node라도 배치해서 실행한다. 
중간에 node의 라벨을 삭제하는 경우는 위와 같다. 

requiredDuringSchedulingRequiredDuringExecution
required는 위와 같으며 
실행중 중간에 node 라벨 삭제하는경우 pod가 퇴출당한다. 



