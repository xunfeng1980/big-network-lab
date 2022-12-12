# Big Network Lab
## Install Vagrant
```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant -y
```

## Install K8S
1. the k8s cluster install script fork form https://github.com/akyriako/kubernetes-vagrant-ubuntu
2. modify k8s.gcr.io image to docker.io/gcmirrors and lock k8s version(1.24.4)
3. modify apt mirror to ustc and use local apt-key.gpg(same as google apt-key.gpg)
4. offline k8s image
```shell
vagrant up # wait a monment
vagrant ssh master
    sudo kubectl get nodes -o wide
```
## Install OpenELB
## L2
## BGP
```
vagrant ssh master
    sudo snap install helm --classic
    sudo kubectl create ns openelb-system
    sudo helm repo add kubesphere-test https://charts.kubesphere.io/test
    sudo helm repo update
    sudo  helm install openelb kubesphere-test/openelb -n openelb-system
    sudo kubectl get pods -n openelb-system
    sudo kubectl scale deploy openelb-manager --replicas=2  -n openelb-system
    #NAME                               READY   STATUS      RESTARTS   AGE
    #openelb-admission-create-ggl2r     0/1     Completed   0          30s
    #openelb-admission-patch-frl8r      0/1     Completed   2          30s
    #openelb-manager-78ddbff8bb-9zh5r   1/1     Running     0          30s
    # wait pod running
    sudo kubectl apply -f /vagrant/openelb/conf/
    sudo kubectl apply -f /vagrant/openelb/test/
    sudo kubectl get pods
    #NAME                           READY   STATUS    RESTARTS   AGE
    #bgp-openelb-55bdd695f7-chtdq   1/1     Running   0          17s
    #bgp-openelb-55bdd695f7-xb48m   1/1     Running   0          17s
    
    # wait pod running
    sudo kubectl get svc bgp-svc
    #NAME      TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    #bgp-svc   LoadBalancer   10.99.10.173   172.22.0.2    80:30948/TCP   41s
    exit
vagrant ssh worker-3
    sudo birdc show route all
    # or
    ip route 
    # access EXTERNAL-IP(VIP)
    curl 172.22.0.2
```
### Failover
```shell
vagrant ssh master
    sudo kubectl scale deploy bgp-openelb --replicas=1
    exit
vagrant ssh worker-3
    ip route
    curl 172.22.0.2
    exit
```
replicas=2
```
ip route 
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.22.0.2 proto bird 
        nexthop via 192.168.57.100 dev enp0s8 weight 1 
        nexthop via 192.168.57.101 dev enp0s8 weight 1 
        nexthop via 192.168.57.102 dev enp0s8 weight 1 
172.22.0.3 proto bird 
        nexthop via 192.168.57.101 dev enp0s8 weight 1 
        nexthop via 192.168.57.102 dev enp0s8 weight 1
```
replicas=1
```
ip route 
default via 10.0.2.2 dev enp0s3 proto dhcp src 10.0.2.15 metric 100 
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 
10.0.2.2 dev enp0s3 proto dhcp scope link src 10.0.2.15 metric 100 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
172.22.0.2 proto bird 
        nexthop via 192.168.57.100 dev enp0s8 weight 1 
        nexthop via 192.168.57.101 dev enp0s8 weight 1 
        nexthop via 192.168.57.102 dev enp0s8 weight 1 
172.22.0.3 proto bird 
        nexthop via 192.168.57.101 dev enp0s8 weight 1 
```

# Ref
[BGP](https://zhuanlan.zhihu.com/p/146539405)

[OpenELB BGP](https://openelb.io/docs/getting-started/usage/use-openelb-in-bgp-mode/)

[OpenELB 使用](https://tinychen.com/20220523-k8s-07-loadbalancer-openelb/)

[腾讯云在容器服务TKE上使用LB直通Pod](https://cloud.tencent.com/document/product/457/48793)

