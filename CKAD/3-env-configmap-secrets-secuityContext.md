쿠버네티스의 환경변수 설정. 

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

![[../../resources/Pasted image 20230529185530.png|400]]
값을 지정하는 대신 configMap이나 secret의 사양에 따르게 할 수 있다. 

1번으로 할 경우 env가 많아지면 관리하기가 여려워지게된다.

### *configMap*
이를 configMap으로 중앙관리를 해서 더 쉽게 관리할수있게 한다.
![[../../resources/Pasted image 20230529185813.png|500]]
#### create configMap
`kubectl create configmap`
으로 imperative 하게 생성할 수도 
`kubectl create -f` 
으로 declare하게 생성할수도 있다.

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
--from-literal 옵션은 명령자체에서 key, value 지정하는데 사용된다.

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
*envFrom은 배열이다*

### Secrets
secrets도 configmap 처럼 imperative, declarative한 방법으로 나뉜다. 
`kubectl create secret generic`

![[../../resources/Pasted image 20230529220827.png|500]]
![[../../resources/Pasted image 20230529220851.png|500]]

*기본적으로 secret은 base64 암호화한다.* 따라서 누구나 base64로 decode할 수 있기 때문에 암호화되어있다고 할 수 없긴하다. 하지만 평문으로 저장하는것보단 안전하다.
`echo -n "mysql" | base64`
`echo -n "6=4sdf" | base64 --decode`

```
k get secrets
k describe secrets
k get secret app-secret -o yaml
```

![[../../resources/Pasted image 20230529221129.png|400]]

![[../../resources/Pasted image 20230529221154.png|400]]

*best practice*
secret object가 있는 것들은 source code repository에 not checking in 해야한다 
ETCD에 암호화되어 저장되도록 *encryption at rest*를 활성화시켜야한다.

kubernetes handles secrets
secret은 해당 node의 pod에 필요한 경우에만 node로 전송된다.
kubelet은 tmpfs에 secret을 저장한다. secret이 disk storage에 not written이다. 
secrets에 depend 되어있는 pod가 삭제될때마다 kubelet은 그 local copy data도 같이 지운다. 

## **securityContext**
![[../../resources/Pasted image 20230530171433.png|500]]
![[../../resources/Pasted image 20230530171447.png|500]]
*container security context가 pod보다 더 우선이다.*

what is the user used to execute the sleep process within the ubuntu-sleeper pod?
In the current(default) namespace.
```
k get pod
> ubuntu-sleeper

k exec ubuntu-sleeper -- whoami 
> root

k exec (pod name) -- whoami
위의 명령어로 어떤사용자로 로그인했는지 알려준다. 
```
