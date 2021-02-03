# **Building a Kubernetes Cluster with Kubeadm**
## **Introduction**
A Kubernetes cluster is a powerful tool for managing containers in a highly-available manner. Kubeadm greatly simplifies the process of setting up a simple cluster. In this hands-on lab, you will build your own working Kubernetes cluster using Kubeadm.

## **Solution**
Begin by logging in to the lab servers using the credentials provided on the hands-on lab page:

        ssh cloud_user@PUBLIC_IP_ADDRESS

## **Install Docker on all three nodes**
Do the following on all three nodes:

1. Add the Docker GPG key:


        curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

2. Add the Docker repository:

        sudo add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"

3. Update packages:

        sudo apt-get update
4. Install Docker:

        sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu

5. Hold Docker at this specific version:

        sudo apt-mark hold docker-ce

6. Verify that Docker is up and running with:

        sudo systemctl status docker

Make sure the Docker service status is **active (running)**!

## **Install Kubeadm, Kubelet, and Kubectl on all three nodes**
Install the Kubernetes components by running this on all three nodes:

1. Add the Kubernetes GPG key:

        curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

2. Add the Kubernetes repository:

        cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
        deb https://apt.kubernetes.io/ kubernetes-xenial main
        EOF

3. Update packages:

        sudo apt-get update

4. Install **kubelet**, **kubeadm**, and **kubectl**:

        sudo apt-get install -y kubelet=1.14.5-00 kubeadm=1.14.5-00 kubectl=1.14.5-00

5. Hold the Kubernetes components at this specific version:

        sudo apt-mark hold kubelet kubeadm kubectl

## **Bootstrap the cluster on the Kube master node**
1. On the Kube master node, do this:

        sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    That command may take a few minutes to complete.

2. When it is done, set up the local kubeconfig:

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

    Take note that the **kubeadm init** command printed a long **kubeadm join** command to the screen. You will need that **kubeadm join** command in the next step!

3. Run the following command on the Kube master node to verify it is up and running:

        kubectl version
    This command should return both a **Client Version** and a **Server Version**.

## **Join the two Kube worker nodes to the cluster**

1. Copy the **kubeadm join** command that was printed by the **kubeadm init** command earlier, with the token and hash. Run this command on both worker nodes, but make sure you add sudo in front of it:

        sudo kubeadm join $some_ip:6443 --token $some_token --discovery-token-ca-cert-hash $some_hash

2. Now, on the Kube master node, make sure your nodes joined the cluster successfully:

        kubectl get nodes

3. Verify that all three of your nodes are listed. It will look something like this:

        NAME            STATUS     ROLES    AGE   VERSION
        ip-10-0-1-101   NotReady   master   30s   v1.12.2
        ip-10-0-1-102   NotReady   &lt;none>   8s    v1.12.2
        ip-10-0-1-103   NotReady   &lt;none>   5s    v1.12.2
    Note that the nodes are expected to be in the NotReady state for now.

## **Set up cluster networking with flannel**
1. Turn on iptables bridge calls on all three nodes:

        echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

        sudo sysctl -p

2. Next, run this only on the Kube master node:

        kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
    Now flannel is installed! Make sure it is working by checking the node status again:

        kubectl get nodes
    After a short time, all three nodes should be in the **Ready** state. If they are not all **Ready** the first time you run **kubectl get nodes**, wait a few moments and try again. It should look something like this:

        NAME            STATUS   ROLES    AGE   VERSION
        ip-10-0-1-101   Ready    master   85s   v1.12.2
        ip-10-0-1-102   Ready    &lt;none>   63s   v1.12.2
        ip-10-0-1-103   Ready    &lt;none>   60s   v1.12.2

## **Conclusion**

Congratulations — you've completed this hands-on lab!