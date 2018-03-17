# Kubernetes Cluster with External ETCD for both Kube and Calico & External Load Balncer's


### Topology

* Three Masters-VM's/Bare Metal (can be changed as per requirement, usually odd numbers-n+1) which run the kubernetes control plane, calico control plane and other components such as heapster, kube-scheduler, kube-dashboard etc.
* Three ETCD-VM's/Bare Metal to host etcd for both Kubernetes and Calico (check the calico configuration to explicitly denote the endpoints)
* Two LB Instances
* Nodes (Register the Kubelet on the nodes with the VIP)


### Requirements

* Disable Swap on all the nodes
* Install Docker on all the nodes (manage to get the newer versions)
```
sudo apt-get update && sudo apt-get install docker-ce -y
```

### ETCD

* Install ETCD
* After creating required files in /lib/systemd/system/etcd.service and /lib/systemd/system/etcd-calico.service
* Create a seperate directory for Calico: sudo mkdir -p /var/lib/etcd-calico

##### Enable the etcd service in systemd
```
sudo systemctl daemon-reload
sudo systemctl enable etcd.service
sudo systemctl enable etcd-calico.service
```
##### Start the etcd services
```
sudo systemctl start etcd.service
sudo systemctl start etcd-calico.service
```
##### Validate that etcd is running-health check
```
curl -XGET http://localhost:2379/health
{"health": "true"}
 
curl -XGET http://localhost:2479/health
{"health": "true"}
```

### Kubernetes

* Install Kubeadm

  - Add Repo
    
    ```
    apt-get update && apt-get install -y apt-transport-https
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
    deb http://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    apt-get update
    ```

  - Install Kubernetes (Mention the release)

    ```
    sudo apt-get -y install kubeadm=1.9.3-00 kubectl=1.9.3-00 kubelet=1.9.3-00 kubernetes-cni=0.6.0-00 && sudo apt-mark hold kubeadm kubectl kubelet kubernetes-cni
    ```
  - Edit kube-adm.conf to disable swap settings

    /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

    ```
    Environment="KUBELET_EXTRA_ARGS=--fail-swap-on=false"
    ```
    sudo systemctl daemon-reload
    ```
* initialize Kubeadm with the init file created

   ```
   sudo kubeadm init --config kube-init.yml
   ```
* Copy to the certs

   - Kubeadm above creates /etc/kubernetes/pki/. Copy the certs to the other two masters and place them after creating /etc/kubernetes/pki/

   ```
   total 56
   drwxr-xr-x 2 root root 4096 Mar 13 22:55 .
   drwxr-xr-x 4 root root 4096 Mar 13 22:55 ..
   -rw-r--r-- 1 root root 1237 Mar 13 22:55 apiserver.crt
   -rw------- 1 root root 1679 Mar 13 22:55 apiserver.key
   -rw-r--r-- 1 root root 1099 Mar 13 22:55 apiserver-kubelet-client.crt
   -rw------- 1 root root 1675 Mar 13 22:55 apiserver-kubelet-client.key
   -rw-r--r-- 1 root root 1025 Mar 13 22:55 ca.crt
   -rw------- 1 root root 1679 Mar 13 22:55 ca.key
   -rw-r--r-- 1 root root 1025 Mar 13 22:55 front-proxy-ca.crt
   -rw------- 1 root root 1679 Mar 13 22:55 front-proxy-ca.key
   -rw-r--r-- 1 root root 1050 Mar 13 22:55 front-proxy-client.crt
   -rw------- 1 root root 1679 Mar 13 22:55 front-proxy-client.key
   -rw------- 1 root root 1679 Mar 13 22:55 sa.key
   -rw------- 1 root root  451 Mar 13 22:55 sa.pub
   ```
* Run kubeadm init on the other two masters
* To enable http communication enable insecure_port

  ```
  sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

  - --insecure-port=8080
  - --insecure-bind-address=0.0.0.0
  ```

### Calico

* Use calico and calico-rbac to provision Calico as CNI

### Joining Nodes

* Kubeadm init provides "kubeadm join"
  ```
  kubeadm join --token 7d5263.bfe7f6595bfa9c76 172.16.80.4:6443 --discovery-token-ca-cert-hash sha256:db98ac60bcdfc69761c9d97170c18684aaeb0bf6e10fdfcf4d30a0288f1b5ec9
 
  kubeadm join --token 7d5263.bfc8f7435bfa9c76 172.16.80.2:6443 --discovery-token-unsafe-skip-ca-verification
  ```
* To get the token : openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

* Make sure to point the "server:" to the VIP, not the individual master. Restart Kubelet to enable the changes.
