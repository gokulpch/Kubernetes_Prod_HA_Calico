# Kubernetes_Prod_HA_Calico
Kubernetes Cluster for Production with Calico as CNI

This repo has two different approaches to create a Kubernetes cluster in production environments with High-Availability using the default Kubeadm and Calico/Flannel as CNI.


1. Kubernetes Cluster:
* External ETCD - Calico and Kubernetes
* External LB_Instances
* Heapster and Kubernetes WebUI

![alt text](https://github.com/gokulpch/Kubernetes_Prod_HA_Calico/blob/master/images/external_etcd%26external_lb.png)

2. Kubernetes Cluster:
* ETCD as Docker Containers 
* Nginx as Docker Containers
* Keepalived on hosts hosting Masters
* Heapster and Kubernetes WebUI

![alt text](https://github.com/gokulpch/Kubernetes_Prod_HA_Calico/blob/master/images/etc_pod%26emb_lb.png)


