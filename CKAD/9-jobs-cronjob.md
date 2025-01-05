## **Jobs**
![[../../resources/Pasted image 20230609192931.png|500]]
When the pod is created, it runs a container performs the computation task and exits and the pod goes into a Completed state.
But, It then recreates the container in an attempt to leave it running. 
Again the container performs the required computation task and exits. 
And kubernetes brings it up again. And this continuous to happen until a threshold is reached. -> 쿠버는 항상 그 task가 수행되길 발한다. 그래서 계속 재시작이 되는것이다. 
default 설정이 
restartPolicy: Always 이다. 
![[../../resources/Pasted image 20230609193105.png|444]]
다음과 같이 Never로 설정하면 다시 restart를 하지 않는다.

우리가 원하는 만큼 포드를 많이 만들고 일이 성공적으로 끝나도록 보장할 수 있는 관리자가 필요하다 
-> **Jobs**
replica set은 정해진 pod 개수가 항상 동작하도록 하는것인 반면
Jobs는 주어진 작업을 완료하는데 사용된다. 

![[../../resources/Pasted image 20230609193206.png|500]]
apiVersion: batch/v1
pod의 yaml파일의 spec이 template에 그대로 들어가는 구조이다. 

*Deleting the job will also result in deleting the pods that were created by the job.*

![[../../resources/Pasted image 20230609193504.png|555]]
job이 세개로 자동으로 pod도 세개가 만들어진다. 
하지만 만약 포드가 실패하면 성공할때까지 포드를 만들려고 노력한다. 

![[../../resources/Pasted image 20230609193452.png|555]]
포드를 순차적으로 만드는 대신 병렬적으로 만들 수 도 있다. 
completed될때까지 포드가 생성된다. 

## **CronJobs**
일정을 잡고 주기적으로 job을 실행시킨다. 
![[../../resources/Pasted image 20230609194140.png|555]]

![[../../resources/Pasted image 20230609194152.png|555]]
spec 아래 schdule 설정 
jobTemplate 아래에 job의 spec을 그대로 옮기면 된다. 
그래서 총 spec이 세개가된다. cronjob의 spec, job의 spec, pod의 spec


