+++
title = 'Container orchestration'
date = '2025-09-07T19:08:11-04:00'
weight = 10
draft = false
+++


Container orchestration manages large numbers of containers across multiple hosts. An orchestrator creates containers as demand increases and removes them when demand drops. It also provides fault recovery, load balancing, and simplified application deployment.

Orchestrators are self-healing: if a container fails, the orchestrator starts a replacement automatically.

## Kubernetes lab

Kubernetes schedules pods on worker nodes. Each worker node requires at least 2 CPUs or 1 CPU with 2 cores, and all nodes must have static IP addresses.

Kubernetes supports addons that extend its core functionality:

- `dns`: Configures name resolution within the cluster.
- `storage`: Sets up a host path to persist data across container restarts.
- `gpu`: Enables NVIDIA GPU access within containers.

### Manual setup

This setup uses one controller node and two worker nodes, though you can add additional worker nodes as needed.

Kubernetes requires several components installed on each node:

- `containerd`: The container runtime that Kubernetes uses to run containers in the cluster.
- `kubeadm`: Bootstraps the cluster — creates a new cluster, adds nodes, and manages certificates.
- `kubectl`: The Kubernetes command-line tool for managing cluster resources.
- `kubelet`: An agent that runs on each node and facilitates communication between nodes via the API.

Kubernetes system pods run in the `kube-system` namespace and provide core cluster functionality.

**Flannel** is a popular Container Network Interface (CNI) plugin that creates a software-defined network (SDN) enabling pod-to-pod communication across nodes.

#### containerd setup

Run these steps on every node in the cluster:

1. Install the package:
   ```bash
   apt install containerd -y
   ```
2. Verify the service is running:
   ```bash
   systemctl status containerd
   ```
3. Create the configuration directory and generate the default config:
   ```bash
   mkdir /etc/containerd
   containerd config default | sudo tee /etc/containerd/config.toml
   ```
4. Open the config file and set the cgroup driver to systemd:
   ```bash
   vim /etc/containerd/config.toml
   ```
   Set:
   ```bash
   SystemdCgroup = true
   ```
5. Disable swap. Kubernetes will not start if swap is enabled:
   ```bash
   sudo swapoff -a
   free -m
   ```
6. Comment out the swap entry in `/etc/fstab` to persist the setting across reboots:
   ```bash
   vim /etc/fstab
   ```
7. Enable IP forwarding in `/etc/sysctl.conf`:
   ```bash
   vim /etc/sysctl.conf
   ```
   Set:
   ```bash
   net.ipv4.ip_forward=1
   ```
8. Create a kernel modules config file to load `br_netfilter` at boot. This module assists with pod networking:
   ```bash
   vim /etc/modules-load.d/k8s.conf
   ```
   Add:
   ```bash
   br_netfilter
   ```
9. Reboot to apply all changes:
   ```bash
   reboot
   ```

#### Kubernetes setup

Run these steps on every node. See the [kubectl installation docs](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/) for the current GPG keyring steps:

1. Add the Kubernetes repository key:
   ```bash
   curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg \
       https://packages.cloud.google.com/apt/doc/apt-key.gpg
   ```
2. Set permissions on the keyring:
   ```bash
   sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   ```
3. Add the Kubernetes repository:
   ```bash
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
       https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' \
       | sudo tee /etc/apt/sources.list.d/kubernetes.list
   sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
   ```
4. Update the package index and install the required packages:
   ```bash
   apt update
   apt install kubeadm kubectl kubelet
   ```

#### Controller setup

Run these steps on the controller node only:

1. Initialize the cluster. Replace the endpoint IP with your controller's address:
   ```bash
   kubeadm init --control-plane-endpoint=192.168.122.200 \
       --node-name controller \
       --pod-network-cidr=10.244.0.0/16
   ```
2. Configure `kubectl` access using the commands printed by `kubeadm init`:
   ```bash
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```
3. Install the Flannel CNI plugin:
   ```bash
   kubectl apply -f \
       https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
   ```

#### Worker node setup

Run the join command on each worker node to add it to the cluster. The join command is printed by `kubeadm init` on the controller:

```bash
sudo kubeadm join <controller-ip-addr>:6443 --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

If you lose the join command, regenerate it on the controller:

```bash
kubeadm token create --print-join-command
```

### kubectl commands

Common commands for managing cluster resources. `k` is the standard alias for `kubectl`:

```bash
k apply -f manifest.yaml                    # apply a manifest file

k get pods                                  # list pods in the default namespace
k get pods -n <namespace>                   # list pods in a specific namespace
k get pods --all-namespaces                 # list pods across all namespaces
k get pods -o wide                          # list pods with additional details

k get nodes                                 # list all nodes in the cluster

k get service                               # list all services

k delete pod <pod-name>                     # delete a pod
k delete service <service-name>             # delete a service
```

### MicroK8s

MicroK8s is a lightweight Kubernetes distribution available as a snap package. It ships with its own version of `kubectl` scoped to the MicroK8s installation. Use `microk8s kubectl` to target your MicroK8s cluster instead of any other Kubernetes context.

To install and verify MicroK8s:

```bash
snap install microk8s --classic                 # install MicroK8s
microk8s                                        # verify installation
microk8s kubectl get all --all-namespaces       # view all components in all namespaces
microk8s enable dns                             # enable the DNS addon
```

### Deploying containers

Labels identify pods so you can reference them in services and other resources. Assign a name to a container port so you can reference it by name instead of number in later manifests.

[linuxserver.io](https://linuxserver.io) maintains images for Linux distributions that run on all architectures.

By default, pods are only reachable within the cluster network (`10.244.x.x`). To expose a pod to external traffic, create a NodePort service. A NodePort maps a port on a pod to a port on the node, making the pod accessible from outside the cluster. NodePort values must be in the range `30000–32767`, and the mapping applies cluster-wide.

#### MicroK8s commands

When using MicroK8s, prefix all `kubectl` commands with `microk8s`. To copy local manifest files into the MicroK8s VM:

```bash
multipass transfer <file.yml> microk8s-vm:          # copy a local manifest into the MicroK8s VM
microk8s kubectl <command>                          # run any kubectl command against MicroK8s
```

#### Pod manifest

This manifest creates an nginx pod using a [linuxserver.io](https://linuxserver.io) image and names the container port for use in the NodePort service:

```yaml
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
```

#### NodePort manifest

This manifest creates a NodePort service that maps port 80 on the pod to port 30080 on the node. The `selector` targets pods with the `app: nginx` label:

```yaml
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

Verify the service is running:

```bash
k get service
```

## Docker 


### Checking running containers

`docker ps` lists all running containers. Each row shows the container ID, image, command, age, status, exposed ports, and name. Use this as your first check when a service is not responding:

```bash
docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED       STATUS       PORTS                NAMES
a3f2b1c4d5e6   nginx:latest   "/docker-entrypoint.…"   2 hours ago   Up 2 hours   0.0.0.0:80->80/tcp   web
```

To include stopped containers, add `-a`. This is useful when a container has exited unexpectedly and you need to inspect its last state or retrieve logs:

```bash
docker ps -a
CONTAINER ID   IMAGE          COMMAND      CREATED       STATUS                    PORTS   NAMES
a3f2b1c4d5e6   nginx:latest   "/docker…"   2 hours ago   Up 2 hours                80/tcp  web
b7c8d9e0f1a2   alpine         "/bin/sh"    3 hours ago   Exited (1) 2 hours ago            worker
```

The exit code in the `STATUS` column tells you how the container stopped. Exit code `0` means it finished cleanly. Any non-zero code indicates an error — check the logs with `docker logs <name>`.

To print only container names, useful in scripts or when many containers are running:

```bash
docker ps --format "{{.Names}}"
web
worker
```

Common format fields:

| Field         | Description               |
| :------------ | :------------------------ |
| `{{.ID}}`     | Shortened container ID    |
| `{{.Image}}`  | Image name and tag        |
| `{{.Status}}` | Running status and uptime |
| `{{.Ports}}`  | Port mappings             |
| `{{.Names}}`  | Container name            |

## Docker Compose

{{< admonition "Lab environment" tip >}}
This section shows the instructions to set up a lab with Docker Compose. You can quickly set it all up with these steps:
1. Clone https://github.com/dolevf/Black-Hat-Bash
2. `cd ./Black-Hat-Bash/lab`
3. `sudo make init`
{{< /admonition>}}

Docker Compose manages multiple Docker containers through a single YAML file called a _Compose file_. Each container, its network, and its volumes are defined once and brought up or torn down with a single command.

### Install Docker and Docker Compose

1. Install prerequisite packages. `ca-certificates` lets your system validate TLS certificates when downloading from external servers. `gnupg` processes the cryptographic signature on Docker's key. Without both, the key download and repo validation in step 2 fail silently or with opaque errors:
   ```bash
   sudo apt install ca-certificates curl gnupg -y
   ```

2. Create the keyring directory and download Docker's GPG key. `apt` requires every third-party repository to be signed by a trusted key. Placing the key under `/etc/apt/keyrings/` rather than the legacy `/etc/apt/trusted.gpg.d/` scopes it to a specific repository through the `signed-by` attribute in step 3, a meaningful security boundary when you manage multiple third-party sources. The `chmod a+r` ensures `apt` can read the key after dropping privileges during package operations:
   ```bash
   sudo install -m 0755 -d /etc/apt/keyrings
   curl -fsSL https://download.docker.com/linux/debian/gpg \
       | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
   sudo chmod a+r /etc/apt/keyrings/docker.gpg
   ```

3. Add the Docker apt repository. The `[signed-by=...]` attribute ties packages from this repo to the Docker key specifically. A repo entry without it implicitly trusts all keys in your keyring, which is a supply-chain risk. The shell substitutions make the command portable across architectures and Debian releases without editing:
   ```bash
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
       https://download.docker.com/linux/debian \
       $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
       | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

4. Update the package index and install Docker. Docker Compose v2 ships as a plugin binary rather than a standalone script. Without `docker-compose-plugin`, `docker compose` returns an error even though Docker itself is installed:
   ```bash
   sudo apt update -y
   sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
   ```

5. Enable the Docker service and start it immediately. The `--now` flag combines `enable` and `start` in one command. `enable` alone configures the service to start at boot but leaves it stopped this session. `start` alone runs it now but not after a reboot:
   ```bash
   sudo systemctl enable docker --now
   ```

6. Verify the installation. This confirms the plugin binary is installed and the Docker CLI can locate it, surfacing PATH or plugin directory misconfigurations before you attempt to run an actual Compose file:
   ```bash
   docker compose --help
   ```

7. Add your user to the `docker` group to run Docker commands without `sudo`. The `-a` flag appends `docker` to your existing groups. Without it, `usermod` replaces all supplementary groups with only `docker`, locking you out of everything else. `newgrp docker` activates the group change in your current shell without a logout:
   ```bash
   sudo usermod -aG docker $USER
   newgrp docker
   ```
