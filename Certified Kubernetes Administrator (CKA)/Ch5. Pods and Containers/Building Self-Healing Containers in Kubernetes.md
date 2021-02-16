# **Building Self-Healing Containers in Kubernetes**
## Introduction
Kubernetes offers several features that can be used together to create self-healing applications in a variety of scenarios. In this lab, you will be able to practice your skills at using features such as probes and restart policies to create a container application that is automatically healed when it stops working.

## Solution
Log in to the provided lab server using the credentials provided:

    ssh cloud_user@<PUBLIC_IP_ADDRESS>
### **Set a Restart Policy to Restart the Container When It Is Down**
1. Find the pod that needs to be modified:

        kubectl get pods -o wide
2. Take note of the **beebox-shipping-data** pod's IP address.

3. Use the **busybox** pod to make a request to the pod to see if it is working:

        kubectl exec busybox -- curl <beebox-shipping-data_ IP>:8080
    We will likely get an error message.

4. Get the pod's YAML descriptor:

        kubectl get pod beebox-shipping-data -o yaml > beebox-shipping-data.yml
5. Open the file:

        vi beebox-shipping-data.yml
6. Set the restartPolicy to Always:

        spec:
        ...
        restartPolicy: Always
        ...

### **Create a Liveness Probe to Detect When the Application Has Crashed**
1. Add a liveness probe:

        spec:
        containers:
        - ...
            name: shipping-data
            livenessProbe:
            httpGet:
                path: /
                port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
            ...
2. Save and exit the file by pressing Escape followed by **:wq**.

3. Delete the pod:

        kubectl delete pod beebox-shipping-data

4. Re-create the pod to apply the changes:

        kubectl apply -f beebox-shipping-data.yml

5. Check the pod status

        kubectl get pods -o wide
6. If you wait a minute or so and check again, you should see the pod is being restarted whenever the application crashes.

7. Check the http response from the pod again (it will have a new IP address since we re-created it):

        kubectl exec busybox -- curl <beebox-shipping-data_IP>:8080
    If you wish, you can explore and see what happens as the application crashes and the pod is restarted automatically.

## **Conclusion**
Congratulations on successfully completing this hands-on lab!