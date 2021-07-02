# METHODS

- Network Policy
- Dashboard
- Kube-bench 
- RBAC
- NodeRestriction
- Encrypted Etcd secret
- OS Level Security
   - Security Context 
   - Set Container User and Group
   - Force Container Non-Root  (runAsNonRoot: true (on securityContext))
   - Privileged Container (Privileged: false (on securityContext))
   - PrivilegedEscalation (allowPrivilegeEscalation: false (on securityContext))
   - PodSecurityPolicy
- OPA
- Dockerfile (read file system,remove shell commands,add user and group,run non-root)
- Kubesec (static analysis)
- Image Vulnerability Scanning (Clair and Trivy)
- ImagePolicyWebhook (https://dzone.com/articles/kubernetes-image-policy-webhook-explained)
- Falco (Threat detection engine)
- Immutability of container at runtime (startupProbe or securityContext)
- Audit
- AppArmor (Linux kernel security - Host)
- Seccomp
- Reduce Attack Surface

# TIPS

### Verify Kubernetes Binaries
```ruby
VERSION=$(v1.21.2)
curl -LO "https://dl.k8s.io/$VERSION/bin/linux/amd64/kubectl.sha256"
curl -LO "https://dl.k8s.io/$VERSION/bin/linux/amd64/kubelet.sha256"
curl -LO "https://dl.k8s.io/$VERSION/bin/linux/amd64/kube-apiserver.sha256"
echo "$(<kubectl.sha256) kubectl" | sha256sum --check
echo "$(<kubelet.sha256) kubelet" | sha256sum --check
echo "$(<kube-apiserver.sha256) kube-apiserver" | sha256sum --check
```

### Kube-bench
```ruby
kube-bench run --targets master,node,etcd,policies --version 1.20

docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest run --targets=master --version 1.21
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest run --targets=node --version 1.21
```
### add new KUBECONFIG
```ruby
kubectl config set-credentials jane --client-key=jane.key --client-certificate=jane.crt
kubectl config set-context jane --cluster=kubernetes --user=jane
kubectl config view
kubectl config get-contexts
kubectl config use-context jane
```
### Cluster Hardening
```ruby
Manuel Api Request
curl -k https://172.17.0.8:6443 --cacert ca --cert crt --key key

cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
ETCDCTL_API=3 etcdctl --cert /etc/kubernetes/pki/apiserver-etcd-client.crt --key /etc/kubernetes/pki/apiserver-etcd-client.key --cacert /etc/kubernetes/pki/etcd/ca.crt endpoint health
ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key get /registry/secrets/default/mykey
```
### Certificate Info
```ruby
openssl req -nodes -new -x509 -keyout accounts.key -out accounts.crt -subj "/CN=accounts.svc"
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -subject
```
### PodSecurityPolicies
```ruby
--enable-admission-plugins=PodSecurityPolicy
kubectl create clusterrole psp-allow --verb=use --resource=podsecuritypolicies
kubectl create clusterrolebinding psp-allow-bn --clusterrole=psp-allow --serviceaccount:default:default
```
### Dockerfile
```ruby
RUN chmod a-w /etc
RUN rm -rf /bin/*
addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
USER appuser
```
### Trivy
```ruby
docker run ghcr.io/aquasecurity/trivy:latest image nginx:latest
trivy image python:3.4-alpine
trivy image --input ruby-2.3.0.tar
```
### Audit 
```ruby
- --audit-policy-file=/etc/kubernetes/audit/policy.yaml
- --audit-log-path=/etc/kubernetes/audit/logs/audit.log
- --audit-log-maxsize=500
- --audit-log-maxbackup=5
```
### AppArmor
```ruby
apparmor_parser /etc/apparmor.d/apparmor-k8s-deny-write
```
### Seccomp
```ruby
docker run --security-opt seccomp=default.json -d nginx
```
### Falco
```ruby
falco -r my_rule.yaml -M 45

tail -f /var/log/syslog | grep falco

- rule: spawned_process_in_monitor_container
  desc: A process was spawned in the Monitor container
  condition: container.name = "monitor" and evt.type=execve
  output: "%evt.time,%container.id, %container.image,%user.uid,%proc.name"
  priority: NOTICE
```
