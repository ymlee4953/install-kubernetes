# **Discovering Pod Resource Usage with Kubernetes Metrics**
## **Introduction**
Kubernetes metrics allow you to gain insight into a wide variety of data about your Kubernetes applications. You can use these metrics to gain insight into how your compute resources are being used. In this lab, you will have the opportunity to hone your skills by investigating existing pods running in a Kubernetes cluster to determine which ones are using the most CPU.

## **Solution**
Log in to the lab server using the credentials provided:

    ssh cloud_user@<PUBLIC_IP_ADDRESS>
### **Install Kubernetes Metrics Server**
1. Install Kubernetes Metrics Server:

        kubectl apply -f https://raw.githubusercontent.com/linuxacademy/content-cka-resources/master/metrics-server-components.yaml
2. Verify Metrics Server is responsive:

        kubectl get --raw /apis/metrics.k8s.io/
    It may take a few minutes for Metrics Server to become responsive to requests.

### **Locate the CPU-Using Pod and Write Its Name to a File**
1. In the beebox-mobile namespace, determine which pod with the label app=auth is using the most CPU:

        kubectl top pod -n beebox-mobile --sort-by cpu --selector app=auth
    If you get an error message saying metrics are not available, wait a few minutes and then run the command again.

2. Write the name of the pod to a file:

        echo auth-proc > /home/cloud_user/cpu-pod-name.txt

## **Conclusion**
Congratulations on successfully completing this hands-on lab!