도커와 쿠버에서도 마찬가지로 컨테이너가 삭제되면 그거에 해당하는 data도 같이 삭제가된다. 
허나 volume을 설정하면 컨테이너가 삭제되도 volume안의 data는 존재하게 된다. 

볼륨을 설정하는데 여러 방법이 존재한다. 
host path 
볼륨에 생성된 파일은 node의 설정한 디렉토리에 데이터가 저장된다. 
컨테이너에 access하기 위해 볼륨이 생성되면 
컨테이너 내부의 디렉토리에 그 볼륨을 마운트한다. 
![[../../resources/Pasted image 20230611230117.png]]
각 컨테이너에서 볼륨마운트 필드를 사용해 컨테이너 내의 디렉터리 옵션에 데이터볼륨울 마운트한다. 

However it is not recommended for use in a *multi-node cluster.* This is because the PODs would use the /data directory on all the nodes, and expect all of them to be the same and have the same data. Since they are on different servers, they are in fact not the same, unless you configure some kind of external replicated clustered storage solution.

![[../../resources/Pasted image 20230611232559.png|555]]
Kubernetes supports several types of standard storage solutions such as NFS, glusterFS, Flocker, FibreChannel, CephFS, ScaleIO or public cloud solutions like AWS EBS, Azure Disk or File or Google’s Persistent Disk.
For example, to configure an AWS Elastic Block Store volume as the storage or the volume, we replace hostPath field of the volume with awsElasticBlockStore field along with the volumeID and filesystem type. The Volume storage will now be on AWS EBS.

## **Persistent Volume**

![[../../resources/Pasted image 20230611233015.png|555]]
You would like it to be configured in a way that an administrator can create a large pool of storage, and then have users carve out pieces from it as required. That is where Persistent Volumes can help us.

A Persistent Volume is a *Cluster wide pool of storage volumes* configured by an **Administrator**, to be used by users deploying applications on the cluster. The users can now select storage from this pool using Persistent Volume Claims.

![[../../resources/Pasted image 20230611233131.png]]

## **Persistent Volume Claim**
![[../../resources/Pasted image 20230611233329.png]]
we will create a Persistent Volume Claim to make the storage available to a node.

Persistent Volumes and Persistent Volume Claims are two separate objects in the Kubernetes namespace. 
*An Administrator creates a set of Persistent Volumes and a user creates Persistent Volume Claims to use the storage.* 

Once the Persistent Volume Claims are created, Kubernetes binds the Persistent Volumes to Claims based on the request and properties set on the volume.

*Every Persistent Volume Claim is bound to a single Persistent volume.* During the binding process, kubernetes tries to find a Persistent Volume that has sufficient Capacity as requested by the Claim, and any other requested properties such as Access Modes, Volume Modes, Storage Class etc.

![[../../resources/Pasted image 20230611233955.png]]
pvc는 다음과 같이 pv에 1대1관계로 binding된다.
그래서 pv보다 pvc의 용량이 작은 경우로 match 될수도 있다. 하지만 1:1관계기 때문에 pv의 용량이 남아도 다른 pvc가 들어올수 없다. pvc는 pv와 match가 안되면 pending상태로 남는다 match가 되기 전까지 말이다.

![[../../resources/Pasted image 20230611233701.png]]

![[../../resources/Pasted image 20230611234212.png|444]]
*Retain*: By default, it is set to Retain. Meaning the Persistent Volume will remain until it is manually deleted by the administrator. It is not available for re-use by any other claims. 

*Delete*: Or it can be Deleted automatically. This way as soon as the claim is deleted, the volume will be deleted as well. 

*Recycle*: Or a third option is to recycle. In this case the data in the volume will be scrubbed before making it available to other claims.

![[../../resources/Pasted image 20230611234540.png|444]]
The same is true for ReplicaSets or Deployments. Add this to the pod template section of a Deployment on ReplicaSet.

## 실습

*volumes의 name과 volumeMounts의 name이 같아야 적용된다.*
*pv와 pvc를 bind하려면 accessModes가 일치해야한다*
*pv와 pvc가 bind 되면 용량이 bind전에가 pvc : 50 pv:100이였다면 pvc도 100이된다*


우선 pv를 확인하고 
pvc에 pv가 갖고있는 label 이 selector로 꼭 있어야 bound됨
그리고 pv가 갖고있는 storageClassName나 volumMode 도 동일하게 있어야함.
