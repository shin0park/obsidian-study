
label중 어느 label을 가지는 object를 보고싶을경우 
pod라하면 
```
k get po --show-label=true
로 보거나 

k get po --selector (key=value)
그거의 개수를 알고 싶을때 
k get po --selector (key=value) | wc -l
-> 헤더 포함이므로 -1 하거나
k get po --selector (key=value) --no-headers | wc -l
```

## Rollout

새로운 쿠버네티스 배포가 생성되거나 기존 배포가 업데이트될때 마다 새로운 rollout을 생성한다. 
*rollout은 백엔드에서 컨테이너를 배포하는 프로세스이다.* k rollout status로 apply후 rolllout이 생성되어 배포하는 것을 볼 수 있다.
그리고 새 출시 버전이 생성될때마다 새 배포 *revision*이 생성된다. 
k rollout history로 revision history를 볼 수 있다.
단 이때 **--recode** 옵션을 줬었어야 history에 기록되어 조회할 수 있다.
 
```
k rollout status deployment/(deploy name)
-> rollout 조회
k rollout history deployment/(deploy name)
```

기본배포전략 - rolling update
![[../../resources/Pasted image 20230609154240.png|500]]
Remember, if you do not specify a strategy while creating the deployment, it will assume it to be Rolling Update. In other words, RollingUpdate is the default Deployment Strategy.

![[../../resources/Pasted image 20230609155059.png|500]]
You could use the kubectl set image command to update the image of your application. But remember, doing it this way will result in the deployment-definition file having a different configuration. So you must be careful when using the same definition file to make changes in the future.
apply로 변경사항 적용할 수 있는데 - set image를 해서 변경할 수도 있다 그런데 주의점이 실제 yaml파일은 변경이 안되니 이런점을 주의해야한다.

### rollback
업데이트 후 잘못 설정해서 롤백하고 싶을때 롤백이 가능하다.
```
k rollout undo deployment/(deploy name)
```
새로 뜬 파드들이 다 파괴되고 이전 포맷으로 돌아간다. 
To rollback to specific revision we will use the `--to-revision` flag.
으로 원하는 특정 revision으로 undo 가능하다.
```
kubectl rollout history deployment nginx --revision=1
kubectl rollout undo deployment nginx --to-revision=1
```

![[../../resources/Pasted image 20230609161019.png|500]]

- **Using the --revision flag:**

Here the revision 1 is the first version where the deployment was created.
You can check the status of each revision individually by using the **--revision flag**:
```
kubectl rollout history deployment nginx --revision=1
deployment.extensions/nginx with revision #1
```
- **Using the --record flag:**
You would have noticed that the "**change-cause**" field is empty in the rollout history output. We can use the **--record flag** to save the command used to create/update a deployment against the revision number.
```
kubectl set image deployment nginx nginx=nginx:1.17 --record
kubectl set image deployment (deploy name) (container name)=(image) --record
```
