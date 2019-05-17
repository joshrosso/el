# Initializing a Cluster

## Launch AMI

Make sure ports 22 and 6443 are open to 0.0.0.0 in your security group and launch an instance using your SSH keypair and the AMI you created with Packer.

```
aws ec2 run-instances \
--image-id AMI \
--count 1 \
--instance-type m5.large \
--key-name AWS_KEY
```

## Use HAProxy for a load balancer

Normally in production you would use a load balancer. For the purposes of this lab we will install HAProxy on this instance and use it for load balancing.

```
sudo apt-get install haproxy vim
sudo vim /etc/haproxy/haproxy.cfg
```

```
#haproxy.cfg
frontend kubernetes
    bind *:6443
    mode tcp
    option tcplog
    timeout client  1m
    default_backend kubernetes
    acl site_dead nbsrv(kubernetes) lt 1
    tcp-request connection reject if site_dead

backend kubernetes
    mode tcp
    option tcplog
    option log-health-checks
    option redispatch
    log global
    balance roundrobin
    timeout connect 10s
    timeout server 1m
    option httpchk GET /healthz
    http-check expect status 200
    server rserve1 127.0.0.1:6444 check check-ssl verify none
    #server rserve1 IP:6444 check check-ssl verify none
    #server rserve1 IP:6444 check check-ssl verify none
```

Restart HAProxy:

```
sudo systemctl restart haproxy.service
```

## Modify a kubeadm InitConfiguration and ClusterConfiguration

SSH to your new instance and create a kubeadm config file called config.yaml:

```
apiVersion: kubeadm.k8s.io/v1beta1
kind: InitConfiguration
localAPIEndpoint:
  bindPort: 6443
---
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
networking:
  podSubnet: ""
```

You can also use kubeadm to see the defaults:

```
sudo kubeadm config print init-defaults
```

Customize the `kubernetesVersion` to be `v1.14.1`.

Customize the `podSubnet` to be `192.168.0.0/16`.

Customize the `controlPlaneEndpoint` to be the IP address of your instance and port 6443.

Customize the `bindPort` to be `6444`. (This is for our HAProxy setup)

## Initialize the Cluster

Once your configuration file is ready, initialize the cluster.

```
sudo kubeadm init --config config.yaml
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Deploy Calico

```
kubectl apply -f https://docs.projectcalico.org/v3.7/manifests/calico.yaml
```

## Verify the installation

Use `kubectl` to verify your installation.

```
kubectl get nodes
kubectl pods -n kube-system
```

Working example:

```
ubuntu@ip-172-31-39-74:~$ kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
calico-kube-controllers-9b4ff7697-gcg9c   1/1     Running   0          3m1s
calico-node-vnktb                         1/1     Running   0          3m1s
coredns-fb8b8dccf-jbmmn                   1/1     Running   0          3m25s
coredns-fb8b8dccf-t8dtm                   1/1     Running   0          3m25s
etcd-ip-172-31-39-74                      1/1     Running   0          2m30s
kube-apiserver-ip-172-31-39-74            1/1     Running   0          2m24s
kube-controller-manager-ip-172-31-39-74   1/1     Running   0          2m18s
kube-proxy-vl8qb                          1/1     Running   0          3m25s
kube-scheduler-ip-172-31-39-74            1/1     Running   0          2m34s
```