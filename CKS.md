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
- Dockerfile - read file system,remove shell commands,add user and group,run non-root
- Kubesec (static analysis)
- Image Vulnerability Scanning (Clair and Trivy)
- ImagePolicyWebhook (https://dzone.com/articles/kubernetes-image-policy-webhook-explained)
- Falco
- Immutability of container at runtime(startupProbe or securityContext)
- Audit
- AppArmor
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
Anonymous Access
kube-apiserver-anonymous-auth=true|false
--anonymous-auth=false

Insecure Acces
--insecure-port=8080

Manuel Api Request
curl -k https://172.17.0.8:6443 --cacert ca --cert crt --key key

certificate info
openssl req -nodes -new -x509 -keyout accounts.key -out accounts.crt -subj "/CN=accounts.svc"
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -subject

NodeRestriction
--enable-admission-plugins=NodeRestriction

cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep etcd
ETCDCTL_API=3 etcdctl --cert /etc/kubernetes/pki/apiserver-etcd-client.crt --key /etc/kubernetes/pki/apiserver-etcd-client.key --cacert /etc/kubernetes/pki/etcd/ca.crt endpoint health
ETCDCTL_API=3 etcdctl --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/apiserver-etcd-client.crt --key=/etc/kubernetes/pki/apiserver-etcd-client.key get /registry/secrets/default/mykey

PodSecurityPolicies
--enable-admission-plugins=PodSecurityPolicy
kubectl create clusterrole psp-allow --verb=use --resource=podsecuritypolicies
kubectl create clusterrolebinding psp-allow-bn --clusterrole=psp-allow --serviceaccount:default:default
```
### OPA 
```ruby
kubectl create -f https://raw.githubusercontent.com/killer-sh/cks-course-environment/master/course-content/opa/gatekeeper.yaml
violation[{"msg": msg}] {
  image := input.review.object.spec.containers[_].image
  not startswith(image, "docker.io/")
  not startswith(image, "k8s.gcr.io/")
  msg := "not trusted image!"
}
```
### Dockerfile
```ruby
RUN chmod a-w /etc
RUN rm -rf /bin/*
addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
USER appuser
```
### KubeSec 
```ruby
docker run -i kubesec/kubesec:512c5e0 scan /dev/stdin < kubesec-test.yaml
```
### Konftest
```ruby
docker run --rm -v $(pwd):/project openpolicyagent/conftest test deploy.yaml
docker run --rm -v $(pwd):/project openpolicyagent/conftest test Dockerfile --all-namespaces
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
docker run --security-opt apparmor=docker-nginx -d nginx
```
### Seccomp
```ruby
docker run --security-opt seccomp=default.json -d nginx
```
