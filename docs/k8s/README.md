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
|  | enp0s8 | 10.0.2.5 | 08:00:27:ef:b8:8f | radmin | k8sNat |
|------|----|----|-----|------|------|
| kubework2 | enp0s3 | 192.168.2.112 | 08:00:27:05:96:e0 | radmin | bridged |
|  | enp0s8 | 10.0.2.5 | 08:00:27:a9:58:98 | radmin | k8sNat |

### Config
1. Create kubemaster and kubework1 VBox using [ubuntu-20.04.3-live-server-amd64.iso](https://ubuntu.com/download/server)

2. Configure network (see above)
   - Set DHCP server IP based on MAC via above table
```
radmin@kubemaster:~$ sudo hostnamectl set-hostname kubemaster
```

3. Docker (All hosts)
```
radmin@kubemaster:~$ sudo apt-get update
radmin@kubemaster:~$ sudo apt-get install docker.io -y
radmin@kubemaster:~$ sudo apt-get install apt-transport-https curl -y
```
   - Saved kubework1-radmin-snapshot1 on catmini://Users/cat/VirtualBox VMs/kubeWork1

4. Kubernetes (All hosts)
```
radmin@kubemaster:~$ cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list 
> deb https://apt.kubernetes.io/ kubernetes-xenial main
> EOF
deb https://apt.kubernetes.io/ kubernetes-xenial main
radmin@kubemaster:~$ sudo apt-get update
```
   - I had to get kubernetes package key got error otherwise
```
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

5. Kubernetes packages (All hosts)
```
radmin@kubemaster:~$ sudo apt-get install -y kubelet=1.18.1-00
radmin@kubemaster:~$ sudo apt-get install -y kubeadm=1.18.1-00
radmin@kubemaster:~$ sudo apt-get install -y kubectl=1.18.1-00
radmin@kubemaster:~$ sudo apt-mark hold kubelet kubeadm kubectl
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
radmin@kubemaster:~$ mkdir -p $HOME/.kube
radmin@kubemaster:~$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
radmin@kubemaster:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
radmin@kubemaster:~$ kubectl cluster-info
Kubernetes master is running at https://kubemaster:6443
KubeDNS is running at https://kubemaster:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```
   - kubectl cluster-info

10. Pod Network Addon(Calico) (Only on Master node) [Calico Install Document](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-kubernetes-api-datastore-50-nodes-or-less)
```
radmin@kubemaster:~$ curl https://docs.projectcalico.org/manifests/calico.yaml -O
radmin@kubemaster:~$ vi calico.yaml
```
   - Search for CALICO_IPV4POOL_CIDR and add k8s IP subnet
```
- name: CALICO_IPV4POOL_CIDR
  value: "10.0.0.0/16"
```
   - Apply config
```
radmin@kubemaster:~$ kubectl apply -f calico.yaml
```
   - Check Status of pods
```
radmin@kubemaster:~$ kubectl get pods -A
NAMESPACE     NAME                                       READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-6c68d67746-c5xc2   1/1     Running   0          70s
kube-system   calico-node-7j9mx                          1/1     Running   0          70s
kube-system   coredns-66bff467f8-z4264                   1/1     Running   0          73m
kube-system   coredns-66bff467f8-z9n4j                   1/1     Running   0          73m
kube-system   etcd-kubemaster                            1/1     Running   1          73m
kube-system   kube-apiserver-kubemaster                  1/1     Running   1          73m
kube-system   kube-controller-manager-kubemaster         1/1     Running   1          73m
kube-system   kube-proxy-bkpt6                           1/1     Running   1          73m
kube-system   kube-scheduler-kubemaster                  1/1     Running   1          73m
radmin@kubemaster:~$ 
```
   - Check nodes
```
radmin@kubemaster:~$ kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
kubemaster   Ready    master   76m   v1.18.1
radmin@kubemaster:~$ 
```

11. Generate Token to add worker Node(Only on Master node)

```
sudo kubeadm token create --description "kubework1 token"
sudo kubeadm token list
```
   - Get ssh hash
```
radmin@kubemaster:~$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | 
> openssl rsa -pubin -outform der 2>/dev/null |
> openssl dgst -sha256 -hex | sed 's/^.* //'
```
   - Single line
```
radmin@kubemaster:~$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt |  openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
```

12. Join Nodes (Only on Worker nodes) kubemaster:6443
```
sudo kubeadm join --token TOKEN_ID CONTROL_PLANE_HOSTNAME:CONTROL_PLANE_PORT --discovery-token-ca-cert-hash sha256:HASH
```
   - Saved kubework1-radmin-snapshot2 on catmini://Users/cat/VirtualBox VMs/kubeWork1
   - Cloned kubework1 to kubework2 with new MAC addr
   - Started kubework1
   - Started kubework2
   - Update kubework2
   ```
   radmin@kubework1:~$ sudo hostnamectl set-hostname kubework2
   radmin@kubework1:~$ sudo dhclient -r
   radmin@kubework1:~$ sudo dhclient
   radmin@kubework1:~$ sudo kubeadm reset -f
   ```
   - Create new token on kubemaster
   ```
   sudo kubeadm token create --description "kubework2 token"
   sudo kubeadm token list
   ```
   - Run Join Node (above) on kubework2
   ```
   sudo kubeadm join --token cwekvr.nwul72bin71b1wk7 kubemaster:6443 --discovery-token-ca-cert-hash sha256:a68afe46f0e77e2f533eaa8988546639fa9ba03a401ea7a8e5643706b6e743e6
   ```
   - Verify nodes running on kubemaster
   ```   
   radmin@kubemaster:~$ kubectl get nodes
   NAME         STATUS   ROLES    AGE     VERSION
   kubemaster   Ready    master   5h22m   v1.18.1
   kubework1    Ready    <none>   118m    v1.18.1
   kubework2    Ready    <none>   28s     v1.18.1
   radmin@kubemaster:~$ 
   ```
   - Saved kubework2-radmin-snapshot1 on catmini://Users/cat/VirtualBox VMs/kubeWork2

13. Want to run workloads on Master? (Only on Master Node) I rather not do this
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
