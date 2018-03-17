# Kubernetes Cluster with Nginx LB as a Docker Container and ETCD as Docker Container with Host Network


### Topology

 ![alt text] (https://github.com/gokulpch/Kubernetes_Prod_HA_Calico/blob/master/images/etc_pod%26emb_lb.png)

* Three Masters-VM's/Bare Metal (can be changed as per requirement, usually odd numbers-n+1) which run the kubernetes control plane, etcd_docker containers, nginx_lb containers, calico control plane and other components such as heapster, kube-scheduler, kube-dashboard etc.
* Three ETCD docker containers running on all three Master hosts.
* Three Nginx docker containers running on all three Master hosts.
* Keepalived on the hosts with Nginx_lb containers with VIP configuration and monitor
* Nodes (Register the Kubelet on the nodes with the VIP)


### Requirements

* Disable Swap on all the nodes
* Install Docker on all the nodes (manage to get the newer versions)
```
sudo apt-get update && sudo apt-get install docker-ce -y
```
* Disable SELINUX or set it to Permissive mode
* Set "net.ipv4.ip_forward = 1" is sysctl

### Setting up Nginx

* Change the lb-nginx-docker/nginx-lb.conf as required (This is the file used as a volume mounted on to the container created below)
* Create a Nginx docker_container on all the three master hosts using lb-nginx-docker/lb-docker-compose.yaml - Make sure to set the volumes in the configuration before creating the container
  ```
   volumes:
    - /nginx/nginx-lb.conf:/etc/nginx/nginx.conf
  ```

### ETCD Container

* Using docker-compose create etcd docker container using etcd-docker/etcd-docker.yaml on all the three master hosts
  ```
  docker-compose --file etcd/etcd-docker.yaml up -d
  ```
* Change the IP addresses as per your topology 
* Use "etcdctl cluster-health" and "etcdctl member list" inside the etcd_container to check the status

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
* initialize Kubeadm with the init file created on first master

   ```
   sudo kubeadm init --config kubernetes/kubeadm-init.yaml
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

* Use calico and calico-rbac in calico/ to provision Calico as CNI
  ```
  kubectl create/apply -f calico.yaml and set the rbac for calico using calico-rbac
  ```

### Flannel

* Use kube-flannel.yaml in flannel/ to provision flannel as CNI
  ```
  kubectl create/apply -f flannel.yaml
  ```

### Joining Nodes

* Kubeadm init provides "kubeadm join"
  ```
  kubeadm join --token 7d5263.cdc9f6425bfa5c7 172.16.80.4:6443 --discovery-token-ca-cert-hash sha256:db98ac60bcdfc69761c9d97170c18684aaeb0bf6e10fdfcf4d30a0288f1b5ec9
 
  kubeadm join --token 7d5263.cdc9f6425bfa5c7 172.16.80.2:6443 --discovery-token-unsafe-skip-ca-verification
  ```
* To get the token : openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

* Make sure to point the "server:" to the VIP, not the individual master. Restart Kubelet to enable the changes.
  
  In /etc/kubernetes/bootstrap-kubelet.conf and kubelet.conf
  
  ```
  sed -i "s/172.16.80.17:6443/172.16.80.18:16443/g" /etc/kubernetes/bootstrap-kubelet.conf
  sed -i "s/172.16.80.17:6443/172.16.80.18:16443/g" /etc/kubernetes/kubelet.conf
  
  Restart kubelet and docker: systemctl restart docker kubelet
  ```
  

### Install Heapster-Influxdb and WebUI

* Using kube-dashboard/kube-dashboard.yaml and heapster.yaml

  ```
  kubectl apply -f kube-dashboard.yaml
  kubectl apply -f influxdb.yaml
  kubectl apply -f heapster.yaml
  kubectl apply -f heapster_rbac.yaml
  ```
