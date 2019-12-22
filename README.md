# k3d

Quick test of [k3d](https://github.com/rancher/k3d) to create a dockerised k3s lab on my Macbook Pro.

## Prerequisites

- Docker
- kubectl

## Installation

A simple command to install k3d cli:

```bash
$ wget -q -O - https://raw.githubusercontent.com/rancher/k3d/master/install.sh | bash
Preparing to install k3d into /usr/local/bin
Password:
k3d installed into /usr/local/bin/k3d
Run 'k3d --help' to see what you can do with it.
```

Here is the k3d cli usage:

```bash
$ k3d --help
NAME:
   k3d - Run k3s in Docker!

USAGE:
   k3d [global options] command [command options] [arguments...]

VERSION:
   v1.3.4

AUTHORS:
   Thorsten Klein <iwilltry42@gmail.com>
   Rishabh Gupta <r.g.gupta@outlook.com>
   Darren Shepherd

COMMANDS:
     check-tools, ct   Check if docker is running
     shell             Start a subshell for a cluster
     create, c         Create a single- or multi-node k3s cluster in docker containers
     delete, d, del    Delete cluster
     stop              Stop cluster
     start             Start a stopped cluster
     list, ls, l       List all clusters
     get-kubeconfig    Get kubeconfig location for cluster
     import-images, i  Import a comma- or space-separated list of container images from your local docker daemon into the cluster
     help, h           Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --verbose      Enable verbose output
   --timestamp    Enable timestamps in logs messages
   --help, -h     show help
   --version, -v  print the version
```

## Create the lab

So, we are ready to create our first cluster.

### Create a cluster

```bash
$ k3d create --name lab --api-port 6551 --publish 8081:80
INFO[0000] Created cluster network with ID 78b83c99074203a8e15036fe46971e6d4edc3c3292895cc300fd1e94fca4800b
INFO[0000] As of v2.0.0 --port will be used for arbitrary port mapping. Please use --api-port/-a instead for configuring the Api Port
INFO[0000] Created docker volume  k3d-lab-images
INFO[0000] Creating cluster [lab]
INFO[0000] Creating server using docker.io/rancher/k3s:v0.10.0...
INFO[0000] SUCCESS: created cluster [lab]
INFO[0000] You can now use the cluster with:

export KUBECONFIG="$(k3d get-kubeconfig --name='lab')"
kubectl cluster-info
```

Configure kubectl to access the lab cluster.

```bash
$ export KUBECONFIG="$(k3d get-kubeconfig --name='lab')" && kubectl cluster-info
Kubernetes master is running at https://127.0.0.1:6551
CoreDNS is running at https://127.0.0.1:6551/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```

```bash
$ k3d list
+------+-------------------------------+---------+---------+
| NAME |             IMAGE             | STATUS  | WORKERS |
+------+-------------------------------+---------+---------+
| lab  | docker.io/rancher/k3s:v0.10.0 | running |   0/0   |
+------+-------------------------------+---------+---------+
```

```bash
$ kubectl get nodes
NAME             STATUS    ROLES     AGE       VERSION
k3d-lab-server   Ready     master    4m        v1.16.2-k3s.1
```

### Install the metrics server

```bash
git clone https://github.com/kubernetes-incubator/metrics-server.git && kubectl apply -f metrics-server/deploy/1.8+/
```

Wait few minutes befor metrics become available.

## Deploy an app

```bash
kubectl deploy nginx.yml
```

Then nginx is now available at [http://localhost:8081](http://localhost:8081).

## Stop the lab

You can stop the lab cluster and resume it later:

```bash
k3d stop --name lab
```
