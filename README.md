# CKAD
```bash
vi ~/.vimrc
set number
set tabstop=2 
set shiftwidth=2 
set expandtab
```
```bash
alias k="kubectl"
alias kns="kubectl config set-context --current --namespace"
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deploy'
alias kdp="kubectl delete po --grace-period=0 --force"
alias krep="kubectl replace --force --grace-period=0 -f"
alias ksec="oc get secret"
alias kcm="oc get cm"
```
```bash
kubectl config set-context k8s
```
