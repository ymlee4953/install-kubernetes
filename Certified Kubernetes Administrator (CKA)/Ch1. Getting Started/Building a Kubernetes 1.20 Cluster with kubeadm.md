# **Building a Kubernetes 1.20 Cluster with Kubeadm**
## **Introduction**

This lab will allow you to practice the process of building a new Kubernetes cluster. You will be given a set of Linux servers, and you will have the opportunity to turn these servers into a functioning Kubernetes cluster. This will help you build the skills necessary to create your own Kubernetes clusters in the real world.

## **Solution**
Log in to the lab server using the credentials provided:

    ssh cloud_user@<PUBLIC_IP_ADDRESS>

## **Install Packages**

1. Log into the Control Plane Node (Note: The following steps must be performed on all three nodes.).
2. Create configuration file for containerd:

        cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
        EOF

3. Load modules:

        sudo modprobe overlay
        sudo modprobe br_netfilter

4. Set system configurations for Kubernetes networking:

        cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
        EOF

5. Apply new settings:

        sudo sysctl --system

6. Install containerd:

        sudo apt-get update && sudo apt-get install -y containerd

7. Create default configuration file for containerd:

        sudo mkdir -p /etc/containerd

8. Generate default containerd configuration and save to the newly created default file:

        sudo containerd config default | sudo tee /etc/containerd/config.toml

9. Restart containerd to ensure new configuration file usage:

        sudo systemctl restart containerd

10. Disable swap:

        sudo swapoff -a

11. Disable swap on startup in **/etc/fstab**:

        sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

12. Install dependency packages:

        sudo apt-get update && sudo apt-get install -y apt-transport-https curl

13. Download and add GPG key:
        curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

14. Add Kubernetes to repository list:

        cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
        EOF


15. Update package listings:

        sudo apt-get update

16. Install Kubernetes packages:

        sudo apt-get install -y kubelet=1.20.1-00 kubeadm=1.20.1-00 kubectl=1.20.1-00

17. Turn off automatic updates:

        sudo apt-mark hold kubelet kubeadm kubectl
        
Log into both Worker Nodes to perform previous steps.
Initialize the Cluster
Initialize the Kubernetes cluster on the control plane node using kubeadm (Note: This is only performed on the Control Plane Node):
sudo kubeadm init --pod-network-cidr 192.168.0.0/16
Set kubectl access:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
Test access to cluster:
kubectl version
Install the Calico Network Add-On
On the Control Plane Node, install Calico Networking:
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
Check status of Calico components:
kubectl get pods -n kube-system
Join the Worker Nodes to the Cluster
In the Control Plane Node, create the token and copy the kubeadm join command (NOTE:The join command can also be found in the output from kubeadm init command):
kubeadm token create --print-join-command
In both Worker Nodes, paste the kubeadm join command to join the cluster:
sudo kubeadm join <join command from previous command>
In the Control Plane Node, view cluster status:
kubectl get nodes
Conclusion
Congratulations — you've completed this hands-on lab!