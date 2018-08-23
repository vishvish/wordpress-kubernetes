# Scaling Wordpress on Kubernetes: A hacker's guide

### Aim: to run multiple replicas of Wordpress with a shared filesystem, not on GCP or AWS.

## Steps

Create a Kubernetes cluster. I used Hetzner Cloud, for it is cheap, and the marvellous CLI tool at https://github.com/xetys/hetzner-kube to create a three-node cluster. I used CX21-CEPH instances for the nodes, and this was the command I used: `hetzner-kube cluster create --name my-cluster --ha-enabled --worker-count 3 --worker-server-type cx21-ceph`

Once the cluster is ready, run this command to replace your `kubeconfig` with one configured for `mycluster`. Yes, I know, does it really have to destroy the previous `kubeconfig`?

`hetzner-kube cluster kubeconfig wp-cluster`

Now build the Rook cluster. (https://rook.github.io/docs/rook/v0.8/ceph-quickstart.html)

```
kubectl create -f operator.yaml

# verify the rook-ceph-operator, rook-ceph-agent, and rook-discover pods are in the `Running` state before proceeding
kubectl -n rook-ceph-system get pod

kubectl create -f cluster.yaml
kubectl -n rook-ceph get pod

# You should see the pods

NAME                                      READY     STATUS      RESTARTS   AGE
rook-ceph-mgr-a-75cc4ccbf4-t8qtx          1/1       Running     0          24m
rook-ceph-mon0-72vx7                      1/1       Running     0          25m
rook-ceph-mon1-rrpm6                      1/1       Running     0          24m
rook-ceph-mon2-zff9r                      1/1       Running     0          24m
rook-ceph-osd-id-0-5fd8cb9747-dvlsb       1/1       Running     0          23m
rook-ceph-osd-id-1-84dc695b48-r5mhf       1/1       Running     0          23m
rook-ceph-osd-id-2-558878cd84-cnp67       1/1       Running     0          23m
rook-ceph-osd-prepare-minikube-wq4f5      0/1       Completed   0          24m
```

Ceph toolbox is very helpful in inspection. Go to https://rook.github.io/docs/rook/v0.8/toolbox.html for instructions.



create storageclass to provide block storage for mysql

create filesystem to provide shared storage

kubectl create secret generic vishvish-mysql --from-literal=password=XXXXXXXXXXXXXXXXXXXXXX

kubectl create secret generic vishvish-wordpress --from-literal=password=XXXXXXXXXXXXXXXXXXXXXX


create mysql

create wordpress deployment

deploy metallb: `kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.7.3/manifests/metallb.yaml`

create metallb configmap based on hetzner node ips

create wordpress service as LoadBalancer

ensure loadbalancing tries to keep sessions to ip using `sessionAffinity: ClientIP`

php settings need configuring:
```
post_max_size = 64M
upload_max_filesize	= 128M
max_execution_time = 300
```

container may need restarting or apache reloading

NB: have rolled above php changes into vishv/wordpress container on docker hub, also found on github at vishvish/wordpress

Install shed-loads of plugins and themes etc

