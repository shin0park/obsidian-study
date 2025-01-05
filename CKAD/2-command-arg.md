docker file 에서 command, argument를 정의할때 
```
ENTRYPOINT ["sleep"]
CMD ["10"]
```
식으로 정의를 한다. 
`docker run ubuntu-sleeper sleep 10`
을 대신해주는 것이다. 하지만 여기서 sleep 5로 다시 지정해준다면 덮어씌워지긴한다. 
이것을 kubernet pod로 정의를 할 수 있다. 
*같은 dockerfile로 만들어진 image를 사용하는 pod라고 하면 pod 에 정의된 cmd, arg로 덮어씌워질 수 있다.* -> *override*
```
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
    - "sleep"
    - "5000"
```
이런식으로 정의할 수 있다.
**command는 항상 배열이다**
```
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "5000"]
```

```
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep"]
    args: ["5000"]
```
과 동일하다.
기억해야될 것이 
명령을 지정할때 **명령은 항상 첫번째 항목에 있어야한다**
sleep = command
5000 = argument

만약 여기서 5000을 2000으로 바꾸고 싶다 했을때 
여러가지 방법이 존재한다. 
1. edit 
   edit을 할 경우에 바꿀수 없는 속성이라며 에러가 발생되고 수정한 내용은 /tmp/new.yaml 이런식으로 저장되게 된다. 
   그럼 기존꺼 지우고 새로생성해야하는데 replace로 빠르게 할수있다.
   `k replace --force -f /tmp/new.yaml`
2. k get po -o yaml > new.yaml
	으로 기존꺼 지우고 새로 생성- 하지만 위와 마찬가지로 replace로 빠르게 할수있다.
	`k replace --force -f new.yaml`
3.  deploy 를 edit deploy 하위에 pod container spec 이 있기 때문에 

애초에 kubectl run으로 pod 생성시 cmd argument를 지정해줄 수도 있다. kubectl run --help을 쳐보면 알 수 있다. 
`kubectl run webapp-green --image=webapp-color -- --color green`
`kubectl run webapp-green --image=webapp-color --command -- python app.py -- --color green`
