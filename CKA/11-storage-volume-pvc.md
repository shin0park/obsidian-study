## Docker in Storage

![[Pasted image 20240526181146.png|500]]
Docker의 storage 에는 두가지 개념이 존재한다. 
- storage drivers
- volume drivers
### Docker storage and file system
docker가 데이터를 어디에 어떻게 저장하고, 컨테이너의 파일 시스템을 어떻게 관리하는가. 

<Docker가 로컬 파일 시스템에 데이터를 저장하는 방법>
![[Pasted image 20240526181740.png|300]]
시스템에 Docker를 설치하면 /var/lib/docker/에 이 폴더 구조가 생성
`cd /var/lib/docker/` 
- aufs
- containers
- image
- volumes
여러 폴더가 있으며, 도커가 default로 모든 데이터를 저장하는 곳이다. - 데이터라고 하면 Docker 호스트에서 실행되는 이미지 및 컨테이너와 관련된 파일을 의미한다. 
예를 들어 컨테이너와 관련된 모든 파일은 컨테이너 폴더 아래에 저장되고 
이미지와 관련된 파일은 이미지 폴더 아래에 저장된다.
도커 컨테이너에 의해 생성된 모든 볼륨은 볼륨 폴더 아래 

#### Layered Architecture
Docker는 이미지와 컨테이너의 파일을 정확이 어떻게 저장할까?
-> Layered Architecture
![[Pasted image 20240526182647.png|600]]
docker는 이미지를 빌드할 때 계층화된 아키텍처에서 이미지를 빌드한다. 
예를 들어, 
Layer 1은 base Ubuntu 운영 체제이고, 
Layer 2는 모든 APT 패키지를 설치
Layer 3는 Python 패키지
Layer 4는 소스 코드 복사
마지막으로  Layer 5는 이미지의 entry point을 업데이트
이렇게 각각 커맨드마다 다른 레이어를 단계별로 생성하여 빌드한다.

base Ubuntu 이미지를 보면 크기가 약 120MB
설치하는 APT 패키지는 약 300MB이고 나머지 레이어는 작다.

이때, 만약 첫번째 도커파일과 매우 유사한 도커파일이 있을때 이미 존재하는 레이어를 쓰는경우
*새로 빌드하지 않고 기존에 필드한 레이어를 재사용한다.*  

이는 애플리케이션 코드를 업데이트하는 경우에도 적용된다. 
app.py와 같은 애플리케이션 코드를 업데이트할 때마다 Docker는 캐시에서 이전 레이어를 모두 *재사용*하고 *최신 소스 코드를 업데이트* 하여 애플리케이션 이미지를 신속하게 다시 빌드한다. 
따라서 재구축 및 업데이트 중에 많은 시간을 절약할 수 있다.

![[Pasted image 20240526183202.png|600]]
더 크게는 Image Layer, Container Layer와 나눌 수 있다.
#### Image Layer
- Image Layer의 layer들은 **최종 도커 이미지를 형성하기 위해 docker build 커맨드를 실행할 때 생성된다** 
  -> 이는 모두 도커 이미지 레이어이다. 
- 빌드가 완료되면 레이어의 내용을 수정할 수 없고, Readonly 전용이며, 새 빌드를 해야만 수정할 수 있다. 

#### Container Layer
- docker run 커맨드를 사용하여 이 이미지를 기반으로 컨테이너를 실행하면 
  Docker는 이러한 레이어를 기반으로 컨테이너를 만들고 이미지 레이어 위에 쓰기 가능한 새 레이어를 만든다.
- ReadWrite Layer
- 애플리케이션이 작성한 로그파일, 컨테이너가 생성한 임시파일, 사용자가 컨테이너에서 수정한 파일 등 컨테이너가 생성한 데이터를 저장하는데에 사용된다.
- 레이어의 수명은 컨테이너가 살아있는 동안만 지속. 컨테이너 소멸시 모든 변경사항도 함께 소멸

![[Pasted image 20240526183815.png|500]]
같은 이미지를 사용하는 경우, 해당 이미지를 사용하여 생성된 모든 컨테이너는 동일한 image layer를 공유한다.
<위 이미지의 상태일때, app.py 소스 코드를 수정하려면 어떻게 해야할까?>
image layer는 readonly이다. 
docker가 자동으로 readwrite layer에 파일 복사본을 생성하고 해당 레이어에서 파일의 다른 버전으로 수정하게 된다. 
이후의 모든 수정은 이 복사본에서 수행되며 이를 -> *Copy on write mechanism*이라고 한다.

-> 하지만 image layer는 readonly기 때문에, 이미지는 `docker build` 커맨드를 사용하여 이미지를 다시 빌드할때까지는 항상 동일하게 유지된다.

그렇다면 이러한 모든 작업을 담당하는 것은 누구인가? 
Layered Architecture를 유지하고, 레이어 생성, 레이어간 파일 복사 및 쓰기 등도 가능하게 하는 것은 
바로 *Storage Driver* 이다.
따라서 Docker는 Storage Driver 를 사용하여 *Layered Architecture*를 활성화합니다.
#### *Storage Driver* 
- AUFS
- ZFS
- BTRFS
- Device Mapper
- Overlay
- Overlay2

어떤 스토리지 드라이버를 사용할지는 OS에 따라 다르며, Docker는 운영체제에 근거해 사용가능한 최고의 드라이버를 선택한다. 
스토리지 드라이버마다 다른 성능과 안정성 특성을 제공하므로, 애플리케이션과 조직의 요구 사항에 맞는 것을 선택하여 사용해야한다.

### Volumes
컨테이너를 제거하면 어떻게 될까?
컨테이너 레이어에 저장된 모든 데이터도 삭제된다. 
컨테이너 데이터를 영구적으로 저장하기 위해서 볼륨을 추가할 수 있다.

예를 들어 데이터베이스로 작업 중이고 컨테이너에서 생성된 데이터를 보존하려는 경우 
persistent volume을 컨테이너에 추가할 수 있다.
![[Pasted image 20240526184222.png|600]]
`docker volume create data_volume` 커맨드를 실행하면 
`var/lib/docker/volumes` 
디렉토리 아래에 `data_volume`이라는 폴더가 생성

`docker run -v data_volume:/var/lib/mysql mysql`

그런 다음 `docker run` 커맨드를 사용하여 docker 컨테이너를 실행할 때 
이와 같이 *-v* 옵션을 사용하고,
data_volume(볼륨명): 콜론 뒤에 컨테이너 내의 위치를 지정
그럼 새로운 컨테이너 생성하고, 생성한 데이터 볼륨은 컨테이너 안에 마운트한다.
-> 즉, `docker containers readwrite` 레이어 내부에 이 볼륨을 마운트할 수 있다.
따라서 데이터베이스가 작성한 데이터는 docker 호스트에 생성된 볼륨에 저장된다.
컨테이너가 파괴돼도 데이터는 살아있게 된다.

*볼륨을 미리 만들지 않아도*
`docker run -v data_volume:/var/lib/mysql mysql`
명령어 입력시 docker는 자동으로 볼륨을 생성하여 컨테이너에 마운트 한다.

#### Bind Mount
하지만 데이터가 이미 다른 위치에 있다면 어떻게 될까? 
예를 들어 /data의 Docker 호스트에 일부 외부 저장소가 있고
default /var/lib/docker 볼륨 폴더가 아닌 */data 볼륨에 데이터베이스 데이터를 저장*하려 한다면??
-> 마운트하려는 폴더의 전체 경로: `/data/mysql` 일때,
`docker run -v /data/mysql:/var/lib/mysql mysql`
컨테이너를 만들고 `/data/mysql` 폴더를 컨테이너에 마운트한다.
이를 *Bind Mount라* 한다.

따라서 Mount 에는 두 가지 유형이 있다.
- Volume Mount
- Bind Mount
볼륨 마운트는 볼륨 디렉토리에서 볼륨을 마운트하고 바인드 마운트는 Docker 호스트의 모든 위치에서 디렉토리를 마운트한다.

***Tip****
-v 사용하는 것은 오래된 스타일. 
새로운 방법은 --mount 옵션을 사용 
--mount는 각 파라미터를 키와 값 형식으로 지정할 수 있으므로 선호되는 방법. 

```shell
$ mkdir -p /data/mysql

$ docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
```


### Volume Driver Plugins in Docker
스토리지 드라이버는 이미지 및 컨테이너의 저장소를 관리하는 데 도움이 된다.
앞서 스토리지를 유지하려면 볼륨을 생성해야 한다는 것을 언급했는데,
*볼륨은 스토리지 드라이버에 의해 처리되지 않는다*

![[Pasted image 20240526181146.png|500]]
*볼륨은 볼륨 드라이버에 의해 처리된다.*
*default 볼륨 드라이버 플러그인은 로컬이다.*

로컬 볼륨 플러그인은 Docker 호스트에서 볼륨을 생성하고 해당 데이터를 
`var/lib/docker` volumes 디렉토리 아래에 저장

타사 솔루션에 볼륨을 생성할 수 있는 여러 볼륨 드라이버 플러그인이 존재한다.
Azure file storage, Convoy, DigitalOcean, Block Storage, Flocker, Google Compute Persistent Disks, Cluster FS, NetApp, REX-Ray, Portworx, 및 VMware vSphere 스토리지 등.

이러한 볼륨 드라이버 중 일부는 다른 스토리지 공급자를 지원합니다. 
예를 들어 REX-Ray 스토리지 드라이버를 사용하여 AWS EBS, S3, Isilon 및 ScaleIO와 같은 EMC 스토리지 어레이, Google Persistent Disk 또는 OpenStack Cinder에 스토리지를 프로비저닝할 수 있습니다.


![[Pasted image 20240526192136.png|500]]
예를들어 REX-Ray EBS와 같은 특정 볼륨 드라이버를 사용하여 Amazon EBS에서 볼륨을 프로비저닝 할 수 있다.
컨테이너가 생성되고 AWS 클라우드에서 볼륨이 연결되며, 컨테이너가 종료되면 데이터는 클라우드에 안전하게 보관된다.

## CSI
#### Container Runtime Interface
![[Pasted image 20240526193909.png|500]]
과거 쿠버네티스는 컨테이너 런타임 엔진으로 도커만 사용했고, 도커와 연동되는 모든 코드는 쿠버네티스 소스 코드에 builtin 되어 있었지만, Rocket 및 CRI-O와 같은 다른 컨테이너 런타임이 등장함에 따라 
다른 컨테이너 런타임으로 작업할 수 있도록 지원을 개방하고 확장하는 것이 중요해졌다. 
이것이 컨테이너 런타임 인터페이스가 탄생한 배경이다.

*CRI는 Kubernetes와 같은 오케스트레이션 솔루션이 Docker와 같은 컨테이너 런타임과 통신하는 방법을 정의하는 표준이다.*
따라서 새로운 컨테이너 런타임 인터페이스가 개발되더라도 CRI 표준을 따르기만 하면 된다. 
새로운 컨테이너 런타임은 쿠버네티스 개발팀과 협력하거나 그 소스코드와 결합할 필요 없이 쿠버네티스와 작동할 수 있게 된다.

#### Container Networking Interface
![[Pasted image 20240526194421.png|500]]
다양한 네트워킹 솔루션에 대한 지원을 확장하기 위해 컨테이너 네트워킹 인터페이스가 도입되었고
새로운 네트워킹 공급업체는 CNI 표준을 기반으로 플러그인을 개발하고 솔루션이 Kubernetes와 함께 작동하도록 만들 수 있다.
#### Container Storage Interface
![[Pasted image 20240526194045.png|500]]
*Container Storage Interface는 다중 스토리지 솔루션을 지원*하도록 개발되었다.
CSI를 사용하면 자체 스토리지용 드라이버를 작성하여 Kubernetes와 함께 사용할 수 있다. 
Port Works, Amazon EBS, Azure Desk, Dell EMC Isilon, PowerMax Unity, XtremIO, NetApp, Nutanix, HPE, Hitachi, Pure Storage. 

CSI는 Kubernetes 특정 표준이 아니고 universal 표준이며 
구현되면, 지원되는 플러그인을 가진 저장소 공급업체와 작동하는 컨테이너 오케스트레이션 도구를 허용한다.
현재 Kubernetes Cloud Foundry와 Mesos는 CSI에 탑재되어있다.

CSI는 컨테이너 오케스트레이터가 호출할 일련의 RPC 또는 원격 프로시저 호출을 정의하며 
이러한 호출은 스토리지 드라이버에 의해 구현되어야 한다.
예를 들어 CSI는 파드가 생성되고 볼륨이 필요한 경우, 컨테이너 오케스트레이터가 볼륨 생성 RPC를 호출하여 볼륨 이름과 같은 세부 사항들을 전달해야한다. 
스토리지 드라이버는 이 RPC를 구현하고 해당 요청을 처리하고 스토리지 어레이에서 새 프로비저닝을 수행한 후 결과를 반환해야 한다. 
마찬가지로 컨테이너 오케스트레이터는 볼륨이 삭제될 때, 볼륨 삭제 RPC를 호출해야 하며 스토리지 드라이버는 해당 호출이 이루어질 때 어레이에서 볼륨을 폐기하는 코드를 구현해야한다. 
그리고 규격에는 솔루션에서 수신해야 하는 파라미터, 교환해야 하는 오류 코드가 색상별로 정확히 명시되어있다. 
[https://github.com/container-storage-interface/spec](https://github.com/container-storage-interface/spec)

## Volumes
![[Pasted image 20240526200639.png|500]]
먼저 Docker의 볼륨을 살펴보면, Docker 컨테이너는 본질적으로 *임시적*이므로 짧은 시간 동안만 지속된다.
  데이터 처리가 필요할 때 호출되고 완료되면 폐기된다. 컨테이너 내의 데이터도 마찬가지이다. 데이터는 컨테이너와 함께 파괴된다.
컨테이너에서 처리한 데이터를 유지하기 하고싶다면, 컨테이너가 생성될 때 컨테이너에 볼륨을 연결한다. 
컨테이너에서 처리한 데이터는 이제 이 볼륨에 배치되어 영구적으로 유지되며, 컨테이너가 삭제되더라도 컨테이너에서 생성되거나 처리된 데이터는 그대로 유지된다.

#### And then, how to work in kubernetes?
Docker에서와 마찬가지로 Kubernetes에서 생성된 파드는 본질적으로 *일시적이다.* 
데이터를 처리하기 위해 Pod를 생성한 후 삭제하면 Pod에서 처리한 데이터도 함께 삭제된다. 
그러기에 데이터를 남기기 위해 파드에 볼륨을 연결한다. 
파드에서 생성된 데이터는 이제 볼륨에 저장되며, 파드이 삭제된 후에도 *데이터가 그대로 유지된다.*

![[Pasted image 20240526200445.png]]
1에서 100 사이의 임의의 숫자를 생성하고 /opt/number.out 으로 파일을 작성하는 간단한 파드를 만든다. 
파드가 삭제되도 번호가 저장된 파일은 볼륨에 남아있게 된다.

#### Volume Storage Options
![[Pasted image 20240526200454.png]]
다음 예시에서는 *hostPath* 옵션을 통해 볼륨을 위한 스토리지 스페이스로 호스트에 직접 설정했다.
단일 노드에서 잘 작동하지만 *다중 노드에서는 사용하지 않는 것이 좋다.*
Pod가 모든 노드에서 /data 디렉토리를 사용하고 모두 동일한 데이터이길 기대하기 때문이다.
하지만 이경우에는 모두 다른 서버이기 때문에 같은게 아니게 될 것 이다. 외부적으로 복제된 클러스터 저장소 솔루션을 구성하지 않는 한말이다.

*Kubernetes는 여러 유형의 다양한 스토리지 솔루션을 지원한다.*
NFS, 클러스터 문제, Flocker, 파이버 채널, Ceph FS, scale io 또는 AWS, EBS, Azure 데스크 또는 파일과 같은 퍼블릭 클라우드 솔루션 또는 Google의 Persistent Desk 

예를 들어 AWS Elastic Block Store 볼륨을 볼륨의 스토리지 옵션으로 구성하기 위해 볼륨의 호스트 경로 필드를 볼륨 ID 및 파일 시스템 유형을 입력해준다.
```shell
volumes:
- name: data-volume
  awsElasticBlockStore:
    volumeID: <volume-id>
    fsType: ext4
```


### Persistent Volumes
유저가 많고, 많은 파드를 배포하는 대규모 환경의 경우에
사용자는 모든 파드에 대해 매번 스토리지를 구성해야한다. 그리고 변경 사항이 있을 때마다 자신의 모든 파드에 변경 사항을 적용해야한다.
*따라서 스토리지를 보다 중앙에서 관리하는게 효율적이며, 관리자가 대규모 스토리지 풀을 생성한 다음 사용자가 필요에 따라 풀을 분할하도록 구성이 필요하다. -> persistent volume*

![[Pasted image 20240526200508.png|500]]
*persistent volume은 관리자가 구성한 cluster-wide 스토리지 볼륨 풀로 
클러스터에 애플리케이션을 배포하는 사용자가 사용한다.*
이제 사용자는 *persistent volume claims*을 사용하여 이 풀에서 스토리지를 선택하여 사용할 수 있게 되는 것이다.
 
<persistent volume을 생성>
```shell
kind: PersistentVolume
apiVersion: v1
metadata:
  name: pv-vol1
spec:
  accessModes: [ "ReadWriteOnce" ]
  capacity:
   storage: 1Gi
  hostPath:
   path: /tmp/data
```
- accessModes
  액세스 모드는 읽기 전용 모드인지 읽기/쓰기 모드인지 등 *호스트에 볼륨을 마운트하는 방법을 정의*한다.
  지원되는 값은 ReadOnlyMany, ReadWriteOnce, ReadWriteMany 모드가 있다.
- storage 
  persistent volume에 대해 스케쥴링할 스토리지 양 
- volume type 
  hostPath: 노드의 로컬 디렉토리
  이 옵션은 프로덕션 환경에서 사용되지 않는다.

```shell
$ kubectl create -f pv-definition.yaml

$ kubectl get pv

$ kubectl delete pv pv-vol1
``` 

### Persistent Volumes Claim

관리자는 영구적인 볼륨인 PV를 생성하고, 사용자는 이를 사용하기 위한 PVC를 생성한다.

PVC가 생성되면 
Kubernetes는 request와 volume에 설정된 속성을 기반으로 pv을 claims에 바인딩한다.

![[Pasted image 20240526203307.png|500]]

![[Pasted image 20240526203248.png|500]]
- 모든 pvc는 단일 pv에 바인딩된다. 1:1
바인딩 프로세스 중에 Kubernetes는 claims에서 요청한 대로 용량이 충분한 pv을 찾으려고 한다. 
- 여기에는 액세스 모드, volume 모드, 스토리지 클래스 등과 같은 기타 모든 요청 속성도 반영된다. 
- *selector* -> 그러나 단일 claims에 대해 가능한 볼륨이 여러 개 있고 특히 특정 volume을 사용하려는 경우엔 레이블과 selector를 사용하여 올바른 volume에 바인딩할 수 있다. 
- 마지막으로, 다른 모든 기준이 일치하고 더 나은 옵션이 없는 경우, 더 작은 claims이 더 큰 volume에 바인딩될 수 있다.

claims과 volume 사이에는 *1:1* 관계가 있으므로 
다른 claims은 volume의 나머지 용량을 사용할 수 없다. 
사용 가능한 volume이 없는 경우, PVC는 클러스터에서 새 volume을 사용할 수 있을 때까지 보류 상태로 유지되며, 
volume을 사용할 수 있게 될때 claims은 자동으로 새로 사용 가능한 volume에 바인딩된다.

<PVC 생성>
![[Pasted image 20240526203234.png|500]]
 claim이 pending상태인 것을 확인할 수 있다. 
조건이 맞는 pv 를 생성하면, pvc가 bound됨을 확인 할수 있따.
![[Pasted image 20240526203417.png|500]]
액세스 모드가 일치하고, 요청된 용량은 500MB이며 volume은 1GB의 스토리지로 구성되어있다.
따라서 pvc는 해당 pv에 바인딩된다.

![[Pasted image 20240526203446.png|500]]
![[Pasted image 20240526203453.png|500]]

pvc가 삭제되면 pv은 어떻게 되는가?
 1. `persistentVolumeReclaimPolicy: Retain` 
    관리자가 수동으로 삭제할 때까지 남아 있음을 의미하며, default로는 볼륨을 유지하도록 설정되어 있다.

2. `persistentVolumeReclaimPolicy: Delete` 
   claims이 삭제되는 즉시 volume도 삭제된다. 따라서 저장 장치의 저장 공간을 확보한다.

3. `persistentVolumeReclaimPolicy: Recycle`
   다른 claims에서 사용하기 전에 volume의 데이터를 삭제한다.

![[../../resources/Pasted image 20230611234540.png|300]]
The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.

#### test

edit pod webapp
using volume
```shell
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      path: /var/log/webapp
      type: Directory
```

create pv
```shell
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-log
spec:
  persistentVolumeReclaimPolicy: Retain
  accessModes:
    - ReadWriteMany
  capacity:
    storage: 100Mi
  hostPath:
    path: /pv/log
```

create pvc
```shell
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: claim-log-1
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 50Mi
```

edit pod webapp
```shell
apiVersion: v1
kind: Pod
metadata:
  name: webapp
spec:
  containers:
  - name: event-simulator
    image: kodekloud/event-simulator
    env:
    - name: LOG_HANDLERS
      value: file
    volumeMounts:
    - mountPath: /log
      name: log-volume

  volumes:
  - name: log-volume
    persistentVolumeClaim:
      claimName: claim-log-1
```


### Storage Class

#### Static Provisioning
![[Pasted image 20240526204441.png|500]]
만약 pvc를 구글클라우드 디스크에서 만든다면
여기서 문제는 이 PV가 생성되기 전에 Google Cloud에 디스크를 생성해야 한다는 것이다. 
애플리케이션에 스토리지가 필요할 때마다 먼저 Google Cloud에서 디스크를 수동으로 프로비저닝한 다음 생성한 디스크와 동일한 이름을 사용하여 persistent volume definition 파일을 수동으로 생성해야한다. 
-> static provisioning volume

#### Dynamic Provisioning -> Storage Class
![[Pasted image 20240526204946.png|600]]
애플리케이션이 필요로 할 때 volume이 자동으로 프로비저닝된다면 좋을 것이다. 
이때 *Storage Class*를 사용 할 수 있다. 
스토리지 클래스로 구글 스토리지 같은 프로비저닝을 정의하여 사용할 수 있다.
스토리지 클래스를 사용하면 Google Cloud에서 스토리지를 자동으로 프로비저닝하고 요구사항이 있으면 파드에 연결할 수 있게 된다.

![[Pasted image 20240526205803.png|600]]
Storage class를 사용하면 더 이상 PV 정의가 필요하지 않으며, 스토리지는 스토리지 클래스가 생성될 때 자동으로 생성된다.
PVC가 우리가 정의한 스토리지 클래스를 사용하려면 PVC 정의에 스토리지 클래스 이름을 지정한다.
-> *storageClassName*
 
다음에 PVC가 생성되면 이와 연결된 스토리지 클래스는 정의된 프로비저너를 사용하여 GCP에서 필요한 크기의 새 디스크를 프로비저닝한 다음 pv을 생성하고 PVC를 해당 volume에 바인딩한다.

 *따라서 여전히 PV를 생성한다는 점을 기억하라.*
 더 이상 수동으로 PV를 생성할 필요가 없다는 것이며, 스토리지 클래스에 의해 자동으로 생성되는 것이다. 

![[Pasted image 20240526210131.png|500]] 
앞서 예시로 든 gce이외에도 각각 다양한 프로비저너들이 존재하는데,
이에 각기 다른 매개변수들을 전달할 수 있다.

![[Pasted image 20240526205935.png|500]]
타입마다 다른 SSD 로 만들 수 있다.

#### test

create pvc
```shell
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 500Mi
```


create pod
using pvc 
```shell
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
      - name: local-persistent-storage
        mountPath: /var/www/html
  volumes:
    - name: local-persistent-storage
      persistentVolumeClaim:
        claimName: local-pvc
```

create storage class
```shell
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: delayed-volume-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```