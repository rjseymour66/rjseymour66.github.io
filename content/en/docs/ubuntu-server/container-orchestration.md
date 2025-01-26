---
title: "Container orchestration"
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
- 

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