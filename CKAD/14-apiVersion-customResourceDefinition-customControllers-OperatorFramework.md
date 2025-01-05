
alpha: 최초 배포된 버전
beta: 말그대로 beta 피드백 제공을 위한 사용자에게 위한
v1: 사용가능한 버전
![[../../resources/Pasted image 20230617233415.png|444]]

deprecation: 나중에는 쓰이지 않게 될 것이라는 뜻
![[../../resources/Pasted image 20230617233430.png]]

![[../../resources/Pasted image 20230617233453.png]]

![[../../resources/Pasted image 20230617233802.png]]

![[../../resources/Pasted image 20230617233515.png]]

![[../../resources/Pasted image 20230617233554.png]]
![[../../resources/Pasted image 20230617233706.png|555]]
![[../../resources/Pasted image 20230617233622.png|555]]


```
kubectl-convert -f (old file) --output-versioon (new-api)
kubectl-convert -f nginx.yaml --output-versioon apps/v1
```
convert 명령어는 별도의 plugin이기 때문ㅇ 설치해야가능하다. 

## **실습**

### 리소스의 api version 보고싶을때 
```
kubectl api-resources
```

In Kubernetes versions : `X.Y.Z`  
Where `X` stands for major, `Y` stands for minor and `Z` stands for patch version.


### Enable the `v1alpha1` version for `rbac.authorization.k8s.io` API group on the `controlplane` node.
->*--runtime-config=*

In case, if anything happens due to misconfiguration you can replace it with the backup file.

```shell
root@controlplane:~# cp -v /etc/kubernetes/manifests/kube-apiserver.yaml /root/kube-apiserver.yaml.backup
```
Now, open up the `kube-apiserver` manifest file in the editor of your choice. It could be `vim` or `nano`.

```shell
root@controlplane:~# vi /etc/kubernetes/manifests/kube-apiserver.yaml
```

Add the `--runtime-config` flag in the `command` field as follows :-

```shell
 - command:
    - kube-apiserver
    - --advertise-address=10.18.17.8
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --runtime-config=rbac.authorization.k8s.io/v1alpha1 --> This one 
```

After that `kubelet` will detect the new changes and will recreate the `apiserver` pod.

It may take some time.

```shell
root@controlplane:~# kubectl get po -n kube-system
```

Check the status of the `apiserver` pod. It should be in running condition.
![[../../resources/Pasted image 20230617234805.png]]

### *Install the `kubectl convert` plugin on the `controlplane` node.*
Download the latest release version from the `curl` command :-

```shell
root@controlplane:~# curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl-convert
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   154  100   154    0     0   6416      0 --:--:-- --:--:-- --:--:--  6416
100 60.7M  100 60.7M    0     0   142M      0 --:--:-- --:--:-- --:--:--  142M
```

then check the availability by `ls` command :-

```shell
root@controlplane:~# pwd
/root
root@controlplane:~# ls
kubectl-convert  multi-pod.yaml  sample.yaml
```

Change the permission of the file and move to the `/usr/local/bin/` directory.

```shell
root@controlplane:~# pwd
/root
root@controlplane:~# chmod +x kubectl-convert 
root@controlplane:~# 
root@controlplane:~# mv kubectl-convert /usr/local/bin/kubectl-convert
root@controlplane:~# 
```

Use the `--help` option to see more option.

```shell
root@controlplane:~# kubectl-convert --help
```

If it'll show more options that means it's configured correctly if it'll give an error that means we haven't set up properly.

### What is the preferred version for 'authoriztion.k8s.io' api group

```
kubectl proxy 8001&

--> starting ~~
curl localhost:8001/apis/authorization.k8s.io
```

## customResourceDefinition (CRD)

custome resource definition을 만들기 위해선 사용자 정의인 CRD를 생성한다음 리소스를 생성해야한다. 
![[../../resources/Pasted image 20230618160820.png|555]]
다음과같이 리소스를 만들면 ETCD에 데이터가 저장되고 개체 상태를 모니터링한다. Controller를 통해 replicaset등 리소스가 유지해야되는 부분을 유지하면서 돌아간다.
![[../../resources/Pasted image 20230618160809.png|555]]
custom도 마찬가지이다.
book.flight라하면 비행사에 예약확인 체크 및의 작업을 해야하기 때문에 *Custom Controller* 가 필요하다.
*컨트롤러는 반복문에서 실행되는 프로세스나 코드이다*
*쿠버네티스 클러스터를 계속 모니터링하고 특정 개체의 변경이벤트에 경청한다*

![[../../resources/Pasted image 20230618160918.png|555]]
custome resource definition을 만들기 위해선 사용자 정의인 CRD를 생성한다음 리소스를 생성해야한다. 
![[../../resources/Pasted image 20230618161002.png|555]]
versions.schema.openAPI3Schema 에 보면 properties들이 나열되어잇는데 
리소스 정의파일에서 spec 부분을 의미한다. 


## 실습

### 1
`crd.yaml` under the `/root` directory. Let’s complete it and create a custom resource definition from it.  
Let’s create a `custom resource definition` called `internals.datasets.kodekloud.com`. Assign the group to `datasets.kodekloud.com` and the resource is accessible only from a specific namespace.  
Make sure the version should be `v1` and needed to enable the version so it’s being served via REST API.  
So finally create a custom resource from a given manifest file called `custom.yaml`.
Note :- Make sure resources should be created successfully from the `custom.yaml` file.
The solution file for crd.yaml is pasted below:

```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: internals.datasets.kodekloud.com 
spec:
  group: datasets.kodekloud.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                internalLoad:
                  type: string
                range:
                  type: integer
                percentage:
                  type: string
  scope: Namespaced 
  names:
    plural: internals
    singular: internal
    kind: Internal
    shortNames:
    - int
```

Also make sure to create custom resource using `kubectl create -f custom.yaml` after correcting and creating CRD.

*spec.group* 리소스생성시 apiVersion을 의미한다. 
*spec.versions.name* apiVersion에서 v1인지 v2인지 주의
*served*  true인지 체크 
*spec.scope* cluster scope일 수도 있고 namespace scope일 수도 있다. 
*spec.names.plural* 복수일때 명이므로 s붙었는지 확인하기 
*spec.names.kind* 앞에 대문자인지확인하기 

### 2
Create a custom resource called `datacenter` and the apiVersion should be `traffic.controller/v1`.  
  
Set the `dataField` length to `2` and `access` permission should be `true`.

```
k get crd
```
로 crd를 확인한다음 
```
k get crd globals.traffic.controller -o yaml
```
으로 속성비교하면서 리소스정의 만들기 

To create a custom resource called `datacenter` :-

```
kind: Global
apiVersion: traffic.controller/v1
metadata:
  name: datacenter
spec:
  dataField: 2
  access: true
```

## Custom Controller

![[../../resources/Pasted image 20230618160809.png|555]]
custom도 마찬가지이다.
book.flight라하면 비행사에 예약확인 체크 및의 작업을 해야하기 때문에 *Custom Controller* 가 필요하다.
*컨트롤러는 반복문에서 실행되는 프로세스나 코드이다*
*쿠버네티스 클러스터를 계속 모니터링하고 특정 개체의 변경이벤트에 경청한다*
custom controller에서 작업하는 여러 custom한 로직들은 직접개발해야된다. 
python보단 go언어로하는게 좋다.

![[../../resources/Pasted image 20230618161845.png|300]] 
![[../../resources/Pasted image 20230618161854.png|300]]
custom controller는 docker image로 말아서 pod에 배포된다

## Operator Framework
![[../../resources/Pasted image 20230618161953.png]]
crd와 cusromcontroller 두 엔터티가 함께 패키지화 되어 단일 엔티티도 배포될 수 있다. 
*operator framework*를 사용하면 

![[../../resources/Pasted image 20230618162004.png]]

![[../../resources/Pasted image 20230618161818.png]]