# **Exploring a Kubernetes Cluster with kubectl**
## **Introduction**
kubectl is the primary interface most people use in order to work with Kubernetes. This lab will give you a chance to test and hone your kubectl skills with a real Kubernetes cluster. You will have the opportunity to collect information and make changes to the cluster, all using kubectl.

## **Solution**
Log in to the provided lab server using the credentials provided:

    ssh cloud_user@<PUBLIC_IP_ADDRESS>

### **Get a List of Persistent Volumes Sorted by Capacity**
1. Get a list of persistent volumes sorted by capacity:

        kubectl get pv

        kubectl get pv -o yaml

        kubectl get pv --sort-by=.spec.capacity.storage
2. Save the list to a file:

        kubectl get pv --sort-by=.spec.capacity.storage > /home/cloud_user/pv_list.txt

3. List the contents of the file to verify it saved:

        cat pv_list.txt

### **Run a Command Inside the quark Pod's Container to Obtain a Key Value**
1. Get the contents of a file from inside the quark pod's container:

        kubectl exec quark -n beebox-mobile -- cat /etc/key/key.txt

2. Save it to a file:

        kubectl exec quark -n beebox-mobile -- cat /etc/key/key.txt > /home/cloud_user/key.txt

3. List the contents of the file to verify it saved:

        cat key.txt

### **Create a Deployment Using a Spec File**
1. Create a deployment using the deployment spec found in home/cloud_user/deployment.yml:

        kubectl apply -f /home/cloud_user/deployment.yml
        kubectl get deployments -n beebox-mobile
        kubectl get pods -n beebox-mobile
### **Delete the beebox-auth Service**
1. Delete beebox-auth-svc:

        kubectl delete service beebox-auth-svc -n beebox-mobile

## **Conclusion**
Congratulations on successfully completing this hands-on lab!