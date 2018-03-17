# Kubernetes Cluster with External ETCD for both Kube and Calico & External Load Balncer's

<<<<<<< HEAD
### Topology
=======
###Topology
>>>>>>> a1040b5ce0470b12a9938838a86ed7862235bc1c
* Three Masters-VM's/Bare Metal (can be changed as per requirement, usually odd numbers-n+1) which run the kubernetes control plane, calico control plane and other components such as heapster, kube-scheduler, kube-dashboard etc.
* Three ETCD-VM's/Bare Metal to host etcd for both Kubernetes and Calico (check the calico configuration to explicitly denote the endpoints)
* Two LB Instances
* Nodes (Register the Kubelet on the nodes with the VIP)

<<<<<<< HEAD
### Requirements
=======
###Requirements
>>>>>>> a1040b5ce0470b12a9938838a86ed7862235bc1c
* Disable Swap on all the nodes
* Install Docker on all the nodes (manage to get the newer versions)
```
sudo apt-get update && sudo apt-get install docker-ce -y
```
<<<<<<< HEAD
### ETCD
=======
###ETCD
>>>>>>> a1040b5ce0470b12a9938838a86ed7862235bc1c

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
