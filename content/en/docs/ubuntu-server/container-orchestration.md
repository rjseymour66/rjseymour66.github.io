---
title: "xContainer orchestration"
# linkTitle: ""
weight: 160
# description:
---

Containers are inexpensive and easy to run, so you might need to manage a large number of containers. Container orchestration helps you make sure containers are running properly:
- easy to scale according to need
- provides tools to recover from faults and load balance
- an orchestrator is a tool that creates containers as demand increases and scales down when no longer needed
- makes app setup easy
- self-healing - if a container goes down, the orchestrator brings one up

## Kubernetes lab

- Schedules pods on worker nodes
- worker node needs at least 2 CPUs or 1 CPU with 2 cores
- Nodes need static IP addresses
- Has addons to extend core package:
  - `dns`: config name resolution
  - `storage`: setup path on host to persist data
  - `gpu` use NVIDIA GPUs within the containers

### Manual setup 

This setup has 1 controller node and 2 worker nodes, but you can have N worker nodes:
- Requires that you setup a container runtime that lets k8s run containers in the cluster
  - suggested runtime is `containerd`
  - `kubeadm`: tools that bootstrap cluster - create new cluster, add node, etc
  - `kubectl`: k8s command line tool
  - `kubelet`: agent that facilitates communication between the nodes with an API
- Kubernetes pods run in `kube-system` namespace. These provide app functionality
- Flannel is a networking layer (sets up a network) - a popular CNI (Container Network Interface) plugin for Kubernetes that provides a software-defined network (SDN) to enable pod-to-pod communication across the nodes in a Kubernetes cluster.

```bash
# --- containerd setup --- #
# Run these commands on each node #
apt install containerd -y                                           # 1. install package
systemctl status containerd                                         # 2. verify status
mkdir /etc/containerd                                               # 3. create config dir
containerd config default | sudo tee /etc/containerd/config.toml    # 4. create config file, send output to stdout
vim /etc/containerd/config.toml                                     # 5. edit config file
SystemdCgroup = true                                                # 6. in config file, set cgroup driver to systemd
sudo swapoff -a                                                     # 7. turn off swap - k8s will abort otherwise
free -m                                                             # 8. confirm swap is all 0s
vim /etc/fstab                                                      # 9. comment out line with swap to persist setting
vim /etc/systctl.conf                                               # 10. open to enable bridging
net.ipv4.ip_forward=1                                               # 11. enable bridging
vim /etc/modules-load.d/k8s.conf                                    # 12. create config file to load kernel module at boot
br_netfilter                                                        # 13. add this line to ../k8s.conf
                                                                    #     assists w/networking
reboot                                                              # 14. reboot when config is complete


# --- K8s setup --- #
# GPG keyring steps: https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/
curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg \        # 1. add repo key so server knows its a trusted source
    https://packages.cloud.google.com/apt/doc/apt-key.gpg

sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg             # 2. let unprivileged apt programs read keyring

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \    # 3. add repo
    https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /'\
    | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list                  # 4. helps tools work correctly
apt update                                                              # 5. update local index
apt install kubeadm kubectl kubelet                                     # 6. install k8s required packages

# --- CONTROLLER ONLY from here --- #
kubeadm init --control-plane-endpoint=192.168.122.200 \                 # 7. Inits a cluster and assigns it a pod network. Add server IP
    --node-name controller \                                            #    server hostname
    --pod-network-cidr=10.244.0.0/16                                    #    internal k8s pod network CIDR

# --- Commands in prev command output --- #
mkdir -p $HOME/.kube                                                    # 8. create local config dir for kubectl
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config                # 9. copy admin file to config dir
sudo chown $(id -u):$(id -g) $HOME/.kube/config                         # 10. change dir ownership to current user

kubectl apply -f \                                                      # 11. install flannel on controller
    https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml

# --- Worker nodes --- #
# run on all worker nodes to join the cluster
sudo kubeadm join <controller-ip-addr>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
kubeadm token create --print-join-command               # regen previous command, if lost
```

### kubectl commands

```bash
k apply -f manifest.yaml

k get pods
k get pods -n <namespace>
k get pods --all-namespaces
k get pods -o wide

k get nodes

k get service

k delete pod <pod-name>
k delete service <service-name>
```

### MicroK8s

Available as a snap package:
- `kubectl`: Kube control - utility to perform actions against your cluster
- `microk8s kubectl`: microk8s own version of the utility that targets your microk8s installation 


```bash
# --- Installation --- #
snap install microk8s --classic                 # 1. install microk8s
microk8s                                        # 2. verify installation
microk8s kubectl get all --all-namespaces       # 3. view all components in all namespaces
microk8s enable dns                             # 4. enable dn addon

```

### Deploying containers

- labels are attached to a pod so you can ID them for tasks
- linuxserver.io maintains images for linux distros to run on all architectures
- assign the port a name to reference it like a var in future commands
- must create NodePort service to expose internal pod network to outside network. For example, a web server is only available from nodes in the cluster on the 10.244.x.x cluster network unless you expose it
  - NodePort exposes a port running on a pod to a port running on a node


```bash
# --- microk8s only --- #
multipass transfer <file.yml> microk8s-vm:          # copy local files into microk8s command
microk8s kubectl <command>                          # prepend all k commands with 'microk8s'

# --- nginx pod --- #
apiVersion: v1                          
kind: Pod
metadata:
  name: nginx-example
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: linuxserver/nginx
      ports:
        - containerPort: 80
          name: "nginx-http"

# --- NodePort manifest to bridge LAN to K8s cluster --- #
# maps port 80 in the pod to 30080 in cluster
# can use ports 30000 - 32767
# selector uses a label on the pod that we want to apply this service to
# port mappings are cluster wide, not just on controller or node running the pod

k get service               # verify service is running
apiVersion: v1
kind: Service
metadata:
  name: nginx-example
spec:
  type: NodePort
  ports:
    - name: http
      port: 80
      nodePort: 30080
      targetPort: nginx-http
  selector:
    app: nginx
```