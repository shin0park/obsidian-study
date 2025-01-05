## Multi Containier Pod
pod 정의 yaml 파일에 spec아래에 containers가 배열이다. 여기에 multi로 container정의하면 된다. 

### Design Patterns
3가지 패턴이 존재한다. 
![[../../resources/Pasted image 20230604170220.png|400]]

1. SIDECAR
   ex) 각각 파드에 이 로그를 수집하는 log수집용 컨테이너를 배포해서 이를 중앙 로그서버에 전송해서 중앙로그서버에 로그를 모은다. 
 2. ADAPTER
    중앙로그서버에 로그를 전송할때 파드마다 로그의 format이 각기 다를것이다. 이를 하나의 format으로 통합해주는 adapter 컨테이너를 함께 배포한다. 중앙로그서버에 전송전 adapter가 format을 통일시켜준다. 
 3. AMBASSADOR
    dev test prod 환경에 따라 연결해야될 db가 다를텐데 이때 ambassador 컨테이너는 proxy역할로 각 맞는 db를 연결해준다. 
    
    
![[../../resources/Pasted image 20230604170433.png|400]]
![[../../resources/Pasted image 20230604170501.png|400]]


### **log 확인**
```
kubectl logs (pod name)

ex)
kubectl logs app -n elastic-stack
kubectl exec -it app -n elastic-stack -- cat /log/app/log
```