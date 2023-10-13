# Setting up a Kubernetes Cluster with WeaveNet Addon

This guide will walk you through the process of setting up a Kubernetes cluster using `kubeadm` and installing the WeaveNet network addon.

## Prerequisites

Before you begin, make sure you have the following prerequisites:

- Two or more Ubuntu servers (Master and Slaves) with a fresh installation.
- SSH access to the servers.
- Root or sudo privileges.

## Installation Steps

### On Both Master and Slave Nodes

1. Update the package list and upgrade installed packages:
    ```bash
    sudo apt update
    ```

2. Disable swap to ensure Kubernetes can manage memory efficiently:
    ```bash
    sudo swapoff -a
    ```

3. Comment out any existing swap entries in the `/etc/fstab` file:
    ```bash
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
    ```

4. Update the package list again:
    ```bash
    sudo apt-get update
    ```

5. Install required packages:
    ```bash
    sudo apt-get install -y apt-transport-https ca-certificates curl
    ```

6. Create a directory for keyrings:
    ```bash
    sudo mkdir -m 755 /etc/apt/keyrings
    ```

7. Add Kubernetes repository keyring:
    ```bash
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    ```

8. Add Kubernetes repository to the sources list:
    ```bash
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```

9. Update the package list again:
    ```bash
    sudo apt-get update
    ```
10. To check available versions of kubectl:
    ```bash
    apt list -a kubectl
    ```

11. To check available versions of kubeadm:
    ```bash
    apt list -a kubeadm
    ```

12. To check available versions of kubelet:
    ```bash
    apt list -a kubelet
    ```

13. Install Kubernetes components (kubelet, kubeadm, containerd) and kubectl (only on Master):
    ```bash
    sudo apt-get install -y kubelet kubeadm containerd kubectl -y
    ```

14. Verify the Kubernetes and kubeadm versions:
    ```bash
    kubectl version
    kubeadm version
    ```
15. Load necessary kernel modules:
    ```bash
    modprobe overlay
    modprobe br_netfilter
    ```

16. Configure sysctl settings for Kubernetes:
    ```bash
    cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
    net.bridge.bridge-nf-call-iptables = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward = 1
    EOF
    sysctl --system
    ```

17. Create a containerd configuration file:
    ```bash
    mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    ```

18. Restart the containerd service:
    ```bash
    systemctl restart containerd
    ```

19. Reload systemd to recognize new units:
    ```bash
    systemctl daemon-reload
    ```

20. Start and enable the kubelet service:
    ```bash
    systemctl start kubelet
    systemctl enable kubelet.service
    ```

21. Check the status of the kubelet service:
    ```bash
    systemctl status kubelet
    ```

### On the Master Node

22. Initialize the Kubernetes cluster:
    ```bash
    kubeadm init
    ```

23. To start using your cluster, run the following commands as a regular user (replace `<master-node-IP>` with your Master Node's IP address):
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

24. Create a token for nodes to join the cluster (use the provided command):
    ```bash
    kubeadm token create --print-join-command
    ```

25. Install the WeaveNet network addon by applying the following YAML file:
    ```bash
    kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
    ```

Now your Kubernetes cluster is set up with the WeaveNet network addon.

For more information on Kubernetes network addons, refer to the official documentation.
