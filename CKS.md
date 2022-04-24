# METHODS

- Network Policy
- Dashboard
- Kube-bench 
- RBAC
- ServiceAccount Token Mounting 
- Restricted Api Access
   - insecure-port=0 (means disable)
   - anonymous-auth=true
   - enable-admission-plugins=NodeRestriction
- Encrypted Etcd secret
- Container Runtime Sandbox (prevent use linux kernal (RuntimeClass))
   - Runtime Katacontainers
   - Runtime gVisor
- OS Level Security
   - Security Context 
   - Set Container User and Group
   - Force Container Non-Root  (runAsNonRoot: true (on securityContext))
   - Privileged Container (Privileged: false (on securityContext))
   - PrivilegedEscalation (allowPrivilegeEscalation: false (on securityContext))
   - PodSecurityPolicy
- OPA
- Dockerfile (read file system,remove shell commands,add user and group,run non-root,multi-stage)
- Kubesec (static analysis)
- Image Vulnerability Scanning (Clair and Trivy)
- ImagePolicyWebhook (https://dzone.com/articles/kubernetes-image-policy-webhook-explained)
- Behavirol Analytics at host and container level
   - strace
   - Falco (Threat detection engine)
- Immutability of container at runtime (startupProbe or securityContext)
- Audit
- Kernal Hardening Tools (Linux kernel security - Host)
   - AppArmor
   - Seccomp
- Reduce Attack Surface

# TIPS

### Verify Kubernetes Binaries
```bash
VERSION=$(v1.21.2)
curl -LO "https://dl.k8s.io/$VERSION/bin/linux/amd64/kubectl.sha256"
curl -LO "https://dl.k8s.io/$VERSION/bin/linux/amd64/kubelet.sha256"
curl -LO "https://dl.k8s.io/$VERSION/bin/linux/amd64/kube-apiserver.sha256"
echo "$(<kubectl.sha256) kubectl" | sha256sum --check
echo "$(<kubelet.sha256) kubelet" | sha256sum --check
echo "$(<kube-apiserver.sha256) kube-apiserver" | sha256sum --check
```

### Kube-bench
```bash
kube-bench run --targets master
kube-bench run --targets master,node,etcd,policies --version 1.20

docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest run --targets=master --version 1.21
docker run --pid=host -v /etc:/etc:ro -v /var:/var:ro -t aquasec/kube-bench:latest run --targets=node --version 1.21
```
### add new KUBECONFIG
```bash
kubectl config set-credentials jane --client-key=jane.key --client-certificate=jane.crt
kubectl config set-context jane --cluster=kubernetes --user=jane
kubectl config view
kubectl config get-contexts
kubectl config use-context jane
```
### Cluster Hardening
```bash
Manuel Api Request
curl -k https://172.17.0.8:6443 --cacert ca --cert crt --key key

cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
ETCDCTL_API=3 etcdctl --cert /etc/kubernetes/pki/apiserver-etcd-client.crt --key /etc/kubernetes/pki/apiserver-etcd-client.key --cacert /etc/kubernetes/pki/etcd/ca.crt endpoint health
ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key get /registry/secrets/default/mykey
```
### Certificate Info
```bash
openssl req -nodes -new -x509 -keyout accounts.key -out accounts.crt -subj "/CN=accounts.svc"
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -subject
```
### PodSecurityPolicies
```bash
--enable-admission-plugins=PodSecurityPolicy
kubectl create clusterrole psp-allow --verb=use --resource=podsecuritypolicies --resource-name=<psp_name>
kubectl create clusterrolebinding psp-allow-bn --clusterrole=psp-allow --serviceaccount:default:default
```
### Dockerfile
```dockerfile
## We'll choose the incredibly lightweight
## Go alpine image to work with
FROM golang:1.13-alpine3.11 as builder
## We create an /app directory in which
## we'll put all of our project code
RUN mkdir /app
ADD . /app
WORKDIR /app
## We want to build our application's binary executable
RUN CGO_ENABLED=1 go build app.go

## the lightweight scratch image we'll
## run our application within
FROM alpine:3.14
COPY --from=builder /app . 
## create user for non root
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
## delete shell commands
RUN rm -rf /bin/*
USER appuser
## we can then kick off our newly compiled
## binary exectuable!!
CMD ["./app"]
```
```bash
RUN chmod a-w /etc
RUN rm -rf /bin/*
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
or
RUN adduser --system --group --no-create-home appuser
USER appuser
```
### Trivy
```bash
docker run ghcr.io/aquasecurity/trivy:latest image nginx:latest
trivy image python:3.4-alpine
trivy image --input ruby-2.3.0.tar
```
### Audit 
```bash
- --audit-policy-file=/etc/kubernetes/audit/policy.yaml
- --audit-log-path=/etc/kubernetes/audit/logs/audit.log
- --audit-log-maxsize=500
- --audit-log-maxbackup=5
```
### AppArmor
```bash
apparmor_parser /etc/apparmor.d/apparmor-k8s-deny-write
```
### Seccomp
```bash
docker run --security-opt seccomp=default.json -d nginx
```
### Falco
```bash
falco -r my_rule.yaml -M 45

tail -f /var/log/syslog | grep falco

- rule: spawned_process_in_monitor_container
  desc: A process was spawned in the Monitor container
  condition: container.name = "monitor" and evt.type=execve
  output: "%evt.time,%container.id, %container.image,%user.uid,%proc.name"
  priority: NOTICE
```
