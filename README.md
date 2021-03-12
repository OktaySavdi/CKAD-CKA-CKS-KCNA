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

Windows: Ctrl+Insert to copy and Shift+Insert to paste.
You can assume elevated privileges on any node by issuing the following command: sudo -i
```
```
https://github.com/dgkanatsios/CKAD-exercises
https://github.com/bbachi/CKAD-Practice-Questions
https://matthewpalmer.net/kubernetes-app-developer/articles/ckad-practice-exam.html
https://codeburst.io/kubernetes-ckad-weekly-challenges-overview-and-tips-7282b36a2681
https://link.medium.com/2p3DbJTQqeb


Exams
https://kubewiz.com/ckad
https://kodekloud.com/p/game-of-pods-game
https://kodekloud.com/courses/675122/lectures/14246972

video
linuxacademy.com - Certified Kubernetes Application Developer (CKAD)
udemy.com        - Kubernetes Certified Application Developer (CKAD)

lab 
https://www.katacoda.com/oktaysavdi

```
