# **Passing Configuration Data to a Kubernetes Container**
## **Introduction**
Kubernetes has multiple options for storing and managing configuration data. This lab will focus on the process of passing that configuration data to your containers in order to configure applications. You will have the opportunity to work with application configuration in Kubernetes hands-on by passing some existing configuration data stored in Secrets and ConfigMaps to a container.

## **Solution**
Log in to the provided lab server using the credentials provided:

    ssh cloud_user@<PUBLIC_IP_ADDRESS>

### **Generate an htpasswd File and Store It as a Secret**
1. Generate an htpasswd file:

        htpasswd -c .htpasswd user

2. Create a password you can easily remember (we'll need it again later).

3. View the file's contents:

        cat .htpasswd
4. Create a Secret containing the htpasswd data:

        kubectl create secret generic nginx-htpasswd --from-file .htpasswd
5. Delete the .htpasswd file:

        rm .htpasswd
### **Create the Nginx Pod**
1. Create pod.yml:

        vi pod.yml
2. Enter the following to create the pod and mount the Nginx config and htpasswd Secret data:

        apiVersion: v1
        kind: Pod
        metadata:
        name: nginx
        spec:
        containers:
        - name: nginx
            image: nginx:1.19.1
            ports:
            - containerPort: 80
            volumeMounts:
            - name: config-volume
            mountPath: /etc/nginx
            - name: htpasswd-volume
            mountPath: /etc/nginx/conf
        volumes:
        - name: config-volume
            configMap:
            name: nginx-config
        - name: htpasswd-volume
            secret:
            secretName: nginx-htpasswd
3. Save and exit the file by pressing Escape followed by :wq.

4. View the existing ConfigMap:

        kubectl get cm
5. Get more info about nginx-config:

        kubectl describe cm nginx-config
6. Create the pod:

        kubectl apply -f pod.yml
7. Check the status of your pod and get its IP address:

        kubectl get pods -o wide
    Its IP address will be listed once it has a Running status. We'll use this in the final two commands.

8. Within the existing busybox pod, without using credentials, verify everything is working:

        kubectl exec busybox -- curl <NGINX_POD_IP>
    We'll see HTML for the 401 Authorization Required page — but this is a good thing, as it means our setup is working as expected.

9. Run the same command again using credentials (including the password you created at the beginning of the lab) to verify everything is working:

        kubectl exec busybox -- curl -u user:<PASSWORD> <NGINX_POD_IP>
This time, we'll see the **Welcome to nginx!** page HTML.

## **Conclusion**
Congratulations on successfully completing this hands-on lab!