# tekton

https://tekton.dev/docs/getting-started/

### Install tekton pipeline, triggers and dashboard:
```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
kubectl apply -f https://github.com/tektoncd/dashboard/releases/latest/download/tekton-dashboard-release.yaml
```

set up port forwarding with Tekton Dashboard:
```bash
kubectl -n tekton-pipelines port-forward svc/tekton-dashboard 9097:9097
```
> Once set up, the dashboard is available in the browser under the address `localhost:9097`

For future use.
create PersistentVolume:
```bash
kubectl apply -f - << EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: tekton
spec:
  storageClassName: manual
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Gi
  hostPath:
    path: "/opt/tekton"
EOF
```
---

### Install tektoncd-cli:
```bash
sudo apt update;sudo apt install -y gnupg
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3EFE0E0A2F2F60AA
echo "deb http://ppa.launchpad.net/tektoncd/cli/ubuntu eoan main"|sudo tee /etc/apt/sources.list.d/tektoncd-ubuntu-cli.list
sudo apt update && sudo apt install -y tektoncd-cli
```

Create a task:
```bash
kubectl apply -f - << EOF
apiVersion: tekton.dev/v1beta1
kind: Task
metadata:
  name: hello
spec:
  steps:
    - name: hello
      image: ubuntu
      command:
        - echo
      args:
        - "Hello World!"
EOF
```

Test dry-run then start the task:  
```bash
tkn task start hello --dry-run
tkn task start hello
```
---

Add TLS certificate
```bash
kubectl create secret tls letsencrypt \
  --key ./shubhamtatvamasi.com.key \
  --cert ./fullchain.cer \
  -n tekton-pipelines
```

Only if you want to expose services to the world. Not recommended for Production. 
Create ingress value: 
```bash
kubectl apply -f - << EOF
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: tekton
  namespace: tekton-pipelines
spec:
  tls:
    - hosts:
      - tekton.k8s.shubhamtatvamasi.com
      secretName: letsencrypt
  rules:
    - host: tekton.k8s.shubhamtatvamasi.com
      http:
        paths:
        - path: /
          backend:
            serviceName: tekton-dashboard
            servicePort: 9097
EOF
```
