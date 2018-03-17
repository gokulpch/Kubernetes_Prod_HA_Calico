# Kubernetes Cluster with External ETCD for both Kube and Calico & External Load Balncer's

####Topology
* Three Masters-VM's/Bare Metal (can be changed as per requirement, usually odd numbers-n+1) which run the kubernetes control plane, calico control plane and other components such as heapster, kube-scheduler, kube-dashboard etc.
* Three ETCD-VM's/Bare Metal to host etcd for both Kubernetes and Calico (check the calico configuration to explicitly denote the endpoints)
* Two LB Instances
* Nodes (Register the Kubelet on the nodes with the VIP)

####Requirements
* Disable Swap on all the nodes
* Install Docker on all the nodes (manage to get the newer versions)
```
sudo apt-get update && sudo apt-get install docker-ce -y
```
####ETCD

#####Enable the etcd service in systemd
```
sudo systemctl daemon-reload
sudo systemctl enable etcd.service
sudo systemctl enable etcd-calico.service
```
#####Start the etcd services
```
sudo systemctl start etcd.service
sudo systemctl start etcd-calico.service
```
#####Validate that etcd is running-health check
```
curl -XGET http://localhost:2379/health
{"health": "true"}
 
curl -XGET http://localhost:2479/health
{"health": "true"}
```
