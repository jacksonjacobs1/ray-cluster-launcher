# Ray Cluster Launcher

## Description
A use case of Ray Cluster Launcher for deploying a Ray cluster on an on-premise cluster.



## Client Prerequisites
This project assumes that your client machine (e.g, personal laptop) has ssh access to two or more on-premise servers.

## Cluster Prerequisites
The on-premise servers must have the following setup:
- Docker is installed.
- The head node firewall exposes ports within the private network. See [here](https://docs.ray.io/en/latest/cluster/cli.html) for default ports.
- The head node has ssh access to all worker nodes.

## Installation
1. Clone this repository
```bash
git clone https://github.com/jacksonjacobs1/ray-cluster-launcher.git
```

2. Change directory to the repository
```bash
cd ray-cluster-launcher
```

3. Crete a virtual environment and activate it
```bash
python3 -m venv venv
source venv/bin/activate
```

4. Install the dependencies
```bash
pip install -r requirements.txt
```

## Cluster Setup
Cluster setup is a user-level procedure, as opposed to a system-level procedure: **Each user must set up their cluster individually.** The client machine is used to launch the cluster, so SSH passwordless login must be enabled between the client machine and all cluster nodes. 

This can be done by generating a public/private key pair on the client machine and copying the public key to the cluster nodes. See [here](https://www.ssh.com/ssh/copy-id) for more details. Here is an example of how this may be done:

1. On the client machine, generate a public/private key pair and press enter through the prompts. It's important to leave the passphrase blank.
```bash
ssh-keygen -t rsa -b 4096 -C "<insert-key-identifier-here>"
```

2. Copy the public key to each node in the cluster.
```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub <head-node-username>@<head-node-ip-address>
ssh-copy-id -i ~/.ssh/id_rsa.pub <worker-node-username>@<worker-node-ip-address>
```

3. Test that passwordless login works.
```bash
ssh -i ~/.ssh/id_rsa <head-node-username>@<head-node-ip-address>
```

## Cluster Configuration
The cluster configuration is defined in the `local_cluster_config.yaml` file. See [here](https://github.com/ray-project/ray/tree/master/python/ray/autoscaler) for more configuration examples. 

You will need to modify the following parameters for your own on-premise cluster:

```yaml
head_ip: <ip-or-hostname-of-head-node>
worker_ips: [<ip-or-hostname-of-worker-node-1>, <ip-or-hostname-of-worker-node-2>, ...]
ssh_user: <username>
min_workers: <number of workers>
max_workers: <number of workers>
```

## Cluster Launch
To launch the cluster, run the following command:
```bash
ray up local_cluster_config.yaml
```

To verify that the cluster is running, attach to the head node:
```bash
ray attach local_cluster_config.yaml
```

Check the status of the cluster:
```bash
(base) ray@SOMAI-SERV01:~$ ray status
======== Autoscaler status: 2023-12-08 14:29:16.383004 ========
Node status
---------------------------------------------------------------
Active:
 2 local.cluster.node
Pending:
 (no pending nodes)
Recent failures:
 (no failures)

Resources
---------------------------------------------------------------
Usage:
 0.0/64.0 CPU
 0.0/4.0 GPU
 0B/203.37GiB memory
 0B/37.92GiB object_store_memory

Demands:
 (no resource demands)
```

## Cluster Teardown
To teardown the cluster, run the following command from the client machine:
```bash
ray down local_cluster_config.yaml
```

Ray down does not always terminate the worker nodes properly, as documented [here](https://github.com/ray-project/ray/issues/11098). Incomplete termination can cause issues while launching subsequent clusters. To check if the cluster has terminated properly, follow these steps:

1. ssh into a worker node
```bash
ssh <worker-node-username>@<worker-node-ip-address>
```

2. Check for a hanging docker container.
```bash
docker ps | grep ray_container
```

3. If the container is still running, stop it.
```bash
docker stop ray_container
```