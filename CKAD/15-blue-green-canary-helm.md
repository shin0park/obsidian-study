canary bluegreen
## blue green deployment

![[../../resources/Pasted image 20230618162637.png|555]]
recreate 전체 다 다운한 다음 새로운 버전 전체 한번에 load하는것
Rolling Update: 이를 하나씩 다운 로드를 반복 -> default 배포방식이다. 
하지만 다른 배포방식으로 구현할 수 있다. 

![[../../resources/Pasted image 20230618162825.png]]
구버전이 새버전과 함께 배포된다. 
구버전이 blue 새버전이 green이다. 
트래픽은 100프로 구버전으로 전송된다 이때 새버전에 대한 테스트가 진행되고 있다. 
모든 테스트가 통과하면 한번에 모든트래픽이 green으로 전환된다. 
이는 *istio 같은 서비스 정책과 가장 잘 구현된다.*

![[../../resources/Pasted image 20230618163129.png]]
새버전의 레이블 버전을 v2로 설정했으니 모든 테스트가 통과되면 *service seletor 아래에 지정된 label만 v2로 변경하면 된다.*

![[../../resources/Pasted image 20230618163320.png]]
![[../../resources/Pasted image 20230618163329.png]]


## Canary

구버전 새버전이 있을때 blue/green처럼 테스트 통과 후 한번에 전환이 아니라 소량씩 트레픽 전환을 하는 방식을 말한다. 
동시에 두버전에 트래픽이 가도록 하길 원하는것이다. 
*istio 같은 서비스 매시가 더나은 제공한다. 각 배포사이에 전송되는 트레픽의 정확한 비율을 정의할 수 있다. 
배포내의 포드 수에 크게 좌우되지 않는다*
![[../../resources/Pasted image 20230618164009.png]]

## 실습
A new deployment called `frontend-v2` has been created in the default namespace using the image `kodekloud/webapp-color:v2`. This deployment will be used to test a newer version of the same app.
Configure the deployment in such a way that the service called `frontend-service` routes less than 20% of traffic to the new deployment.  
Do not increase the replicas of the `frontend` deployment.
`---------------------------------`
The `frontend-v2` deployment currently has 2 replicas. The `frontend` service now routes traffic to 7 pods in total ( 5 replicas on the `frontend` deployment and 2 replicas from `frontend-v2` deployment).
Since the service distributes traffic to all pods equally, in this case, approximately 29% of the traffic will go to `frontend-v2` deployment
  
To reduce this below 20%, scale down the pods on the v2 version to the minimum possible replicas = `1`.
Run: **kubectl scale deployment --replicas=1 frontend-v2**
Once this is done, only ~17% of traffic should go to the v2 version.

## Helm

helm은 package manager 로 작동한다. 설치또는 제거도 마법사와 함께한다. 
릴리즈 manager로 upgrade rollback을 돕는다. 
![[../../resources/Pasted image 20230618174732.png|444]]

### helm install
kubernetes, kubectl, kubconfig 로그인 setup 이 있어야 install 할 수 있다.
`sudo snap install helm --classic`

### Release
```
helm install [release-name] [chart-name]
```
### Command
```
helm list
helm uninstall my-release
helm pull --untar bitnami/wordpress
ls wordpress
helm install release-4 ./wordpress
```


## 실습

*Identify the name of the Operating system installed.*
Run the command
`cat /etc/*release*` 
and identify the name of the operating system.

*Which command is used to search for a `wordpress` helm chart package from the `Artifact Hub`?*
Run `helm search hub chart-name` command to search specific charts on `Artifact Hub`.

*search repo*
`helm search repo joomla`

*How many `helm` repositories are added in the `controlplane` node?*
`helm repo list`

Download the `bitnami apache` package under the `/root` directory.
**Note:** Do not install the package. Just download it.
`helm pull --untar bitnami/apache`