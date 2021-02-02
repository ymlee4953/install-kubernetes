# Managing an Application with Istio

## Introduction
In this lab, we're going to enable monitoring in our Istio mesh, using both Prometheus and Grafana. Once we have the ports forwarded to localhost on the master node, we will need to implement an Nginx proxy to allow outside access.

## The Scenario
Our company has implemented a new SLA for their application that is deployed in Kubernetes. We have been tasked with managing this application.

In order to ensure that support is responsive to issues within the SLA, we will need to implement monitoring of the application so that we can develop some insight into its behavior.

## Getting Logged In
On the lab overview page, we'll see three servers: a Kube Master ( we'll call it km), and two Kube Nodes (kn1 and kn2). We can log into any of them using either an SSH client of our own, or by using Linux Academy's built in Instant Terminal program.

## Install Istio in Kubernetes and Deploy the bookinfo Application
Get the Istio installation package and unpack it:

    [cloud_user@km]$ wget https://github.com/istio/istio/releases/download/1.0.6/istio-1.0.6-linux.tar.gz
    [cloud_user@km]$ tar -xvf istio-1.0.6-linux.tar.gz

Add **istioctl** to our path:

    [cloud_user@km]$ export PATH=$PWD/istio-1.0.6/bin:$PATH

Set Istio to NodePort type, and set the nodePort to port **30080**:

    [cloud_user@km]$ sed -i 's/LoadBalancer/NodePort/;s/31380/30080/' ./istio-1.0.6/install/kubernetes/istio-demo.yaml

Bring up the Istio control plane:

    [cloud_user@km]$ kubectl apply -f ./istio-1.0.6/install/kubernetes/istio-demo.yaml

Verify that the control plane is running, but note that it may take a minute or two:

    [cloud_user@km]$ kubectl -n istio-system get pods

Install the bookinfo application with manual sidecar injection:

    kubectl apply -f <(istioctl kube-inject -f istio-1.0.6/samples/bookinfo/platform/kube/bookinfo.yaml)

Verify that the application is running and that there are 2 containers per pod. This may take another minute or so:

    [cloud_user@km]$ kubectl get pods

Create an ingress and virtual service for the application:

    [cloud_user@km]$ kubectl apply -f istio-1.0.6/samples/bookinfo/networking/bookinfo-gateway.yaml

Now, in a web browser, let's verify the page loads at the URL http://<kn1_PUBLIC_IP ADDRESS>:30080/productpage.

## Configure the Forwarder, Install Nginx, and Access the Grafana Dashboard

Configure Grafana to forward to localhost:

    [cloud_user@km]$ kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000 &

Install and configure Nginx to proxy to the Grafana port:

    [cloud_user@km]$ sudo apt install -y nginx

Change the Nginx config by editing /etc/nginx/sites-enabled/default with whichever text editor you like (but remember privileges, and run something like sudo vi...). Comment out all existing lines in the location / directive, and add proxy_pass http://127.0.0.1:3000. The results should look like this:

    location / {
        # First attempt to serve request as file, then
        # as directory, then fall back to displaying a 404.
        #try_files $uri $uri/ =404;
        proxy_pass http://127.0.0.1:3000;
    }


Then restart Nginx:

    [cloud_user@km]$ sudo systemctl restart nginx

Load Grafana at http://<km_PUBLIC_IP ADDRESS/dashboard/db/istio-mesh-dashboard and review the installed Istio dashboards.

If the mesh traffic dashboard does not populate when the application is refreshed, then rerun this command to re-apply the mixer configuration:

    [cloud_user@km]$ kubectl apply -f ./istio-1.0.6/install/kubernetes/istio-demo.yaml

## Conclusion

Now we can monitor the application that our company wants us keeping an eye on, and we're going it in a pretty slick looking dashboard. Congratulations!