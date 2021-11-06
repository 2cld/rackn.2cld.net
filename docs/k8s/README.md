# Kubernetes notes

## Manual cluster on VBox
- [Configure Multi-node Kubernetes Cluster on VirtualBox](https://www.youtube.com/watch?v=EHDDm_iR1Fs)

### Network
- kubemaster 192.168.2.110 mac 08:00:27:66:8f:9c via https://192.168.2.1
- kubework1  192.168.2.111 mac 08:00:27:e8:10:4c via https://192.168.2.1

| name | if | ip | mac | user | nettype |
|------|----|----|-----|------|------|
| kubemaster | enp0s3 | 192.168.2.110 | 08:00:27:66:8f:9c | radmin | bridged |
|  | enp0s8 | 10.0.2.4 | 08:00:27:da:aa:6a | radmin | k8sNat |
|------|----|----|-----|------|------|
| kubework1 | enp0s3 | 192.168.2.111 | 08:00:27:e8:10:4c | radmin | bridged |
|  | enp0s3 | 10.0.2.5 | 08:00:27:ef:b8:8f | radmin | k8sNat |

### Config
1. Create kubemaster and kubework1 VBox using [ubuntu-20.04.3-live-server-amd64.iso](https://ubuntu.com/download/server)

2. Configure network (see above)

3. Docker (All hosts)
```
radmin@kubemaster:~$ sudo apt-get update
radmin@kubemaster:~$ sudo apt-get install docker.io -y
radmin@kubemaster:~$ sudo apt-get install apt-transport-https curl -y
```

4. Kubernetes (All hosts)
```
radmin@kubemaster:~$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list 
> deb https://apt.kubernetes.io/ kubernetes-xenial main
> EOF
deb https://apt.kubernetes.io/ kubernetes-xenial main
radmin@kubemaster:~$ sudo apt-get update
```

5. Kubernetes packages (All hosts)
```
radmin@kubemaster:~$ sudo apt-get install -y kubelet=1.18.1-00
radmin@kubemaster:~$ sudo apt-get install -y kubeadm=1.18.1-00
radmin@kubemaster:~$ sudo apt-get install -y kubectl=1.18.1-00
radmin@kubemaster:~$ apt-mark hold kubelet kubeadm kubectl
```

6. Add the hosts entry (All hosts) edit the file "/etc/hosts" add "kubemaster 192.168.2.110"

7. Disable SWAP (All hosts)
```
free -m # show current swap
sudo swapoff -a
edit /etc/fstab to remove the swap entry comment out # /swap.img
```

8. Initiate the Cluster(Only on Master node)
```
sudo kubeadm init --control-plane-endpoint kube-master:6443 --pod-network-cidr 10.10.0.0/16
```
   - If kubeadm messes up [kubeadm reset -f](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-reset/)
   - Saved kubemaster-radmin-snapshot1 on catmini://Users/cat/VirtualBox VMs/kubeMaster

9. Set the kubectl context auth to connect to the cluster(Only on Master node)
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

10. Pod Network Addon(Calico) (Only on Master node) [Calico Install Document](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-kubernetes-api-datastore-50-nodes-or-less)
```
curl https://docs.projectcalico.org/manifests/calico.yaml -O
vi calico.yaml
```

11. Generate Token to add worker Node(Only on Master node)

```
sudo kubeadm token create
sudo kubeadm token list
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | 
   openssl rsa -pubin -outform der 2(GREATER THAN SYMBOL)/dev/null | 
   openssl dgst -sha256 -hex | sed 's/^.* //'
```

12. Join Nodes (Only on Worker nodes)
```
sudo kubeadm join --token TOKEN_ID CONTROL_PLANE_HOSTNAME:CONTROL_PLANE_PORT --discovery-token-ca-cert-hash sha256:HASH
```

13. Want to run workloads on Master?(Only on Master Node)
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

14. Sample Deployment file:
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

15. Apply the deployment:
```
kubectl apply -f FILE_NAME
```
