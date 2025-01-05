## alias
```
export ns=default
alias k='kubectl -n $ns' 
# This helps when namespace in question doesn't have a friendly name 
alias kdr= 'kubectl -n $ns -o yaml --dry-run=client'.  
# run commands in dry run mode and generate yaml.

alias k=kubectl
alias ka="kubectl apply -f"
alias kgp="kubectl get po"
alias kgs="kubectl get svc"
alias kga="kubectl get all"
alias kgn="kubectl get node"
alias kgns="kubectl get ns"
alias kgse="kubectl get secrets"
alias kdes="kubectl describe"
```

## vim
 여러줄 한번에 띄어쓰기 
 대문자 V누르면서 마우스로 옮길부분 드래그 -> shift . 치키

## 시작전 보고갈 명령어
### kubeconfig
```
k config view
k config -h 
k config current-context
k config get-context

#user 생성
k config set-credenticals
#cluster 생성
k config set-cluster
#context 생성
k config set-context
#여러 context들중 context 지정
k config use-context (name)

```

### secret
``echo -n "mysql" | base64`
`echo -n "6=4sdf" | base64 --decode`
암복호화 
