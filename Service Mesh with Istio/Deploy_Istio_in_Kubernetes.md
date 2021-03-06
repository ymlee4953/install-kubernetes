# Istio in Kubernetes

##  The Scenario

Our company has an application sitting in a Kubernetes environment, and we need to implement routing rules for it. We will need to log into the Kubernetes environment and deploy Istio. Once Istio is in place, we will deploy the application and verify that routing rules are being applied correctly.

## Getting Logged In
On the lab overview page, we'll see three servers: a Kube Master ( we'll call it km), and two Kube Nodes (kn1 and kn2). We can log into any of them using either an SSH client of our own, or by using Linux Academy's built in Instant Terminal program.

## Deploy Istio into a Kubernetes Cluster and Deploy the bookinfo Application

Get the Istio installation package onto the Kube Master and unpack it:

    [cloud_user@km]$ wget https://github.com/istio/istio/releases/download/1.0.6/istio-1.0.6-linux.tar.gz

    [cloud_user@km]$ tar -xvf istio-1.0.6-linux.tar.gz

Add **istioctl** to our path:

    [cloud_user@km]$ export PATH=$PWD/istio-1.0.6/bin:$PATH

- Check the Istio commnad is available:

    [cloud_user@km]$ istsoctl

Set Istio to NodePort at port 30080:

    [cloud_user@km]$ sed -i 's/LoadBalancer/NodePort/;s/31380/30080/' ./istio-1.0.6/install/kubernetes/istio-demo.yaml

Bring up the Istio control plane:

    [cloud_user@km]$ kubectl apply -f ./istio-1.0.6/install/kubernetes/istio-demo.yaml

Verify that the control plane is running:

    [cloud_user@km]$ kubectl -n istio-system get pods

When all of the pods are up and running (which we can verify by running that command again) we can move on.

Install the bookinfo application with manual sidecar injection:

    [cloud_user@km]$ kubectl apply -f <(istioctl kube-inject -f istio-1.0.6/samples/bookinfo/platform/kube/bookinfo.yaml)

Verify that the application is running and that there are 2 containers per pod:

    [cloud_user@km]$ kubectl get pods

Ignore the **busybox** pod, that's part of the environment.

Once everything is running, let's create an ingress and virtual service for the application:

    [cloud_user@km]$ kubectl apply -f istio-1.0.6/samples/bookinfo/networking/bookinfo-gateway.yaml

Verify the page loads at the uri http://<kn1_IP ADDRESS>:30080/productpage

## Verify That Routing Rules Are Working by Configuring the Application to Route to v1 Then v2 of the reviews Backend Service

Set the default destination rules:

    [cloud_user@km]$ kubectl apply -f istio-1.0.6/samples/bookinfo/networking/destination-rule-all.yaml

Route all traffic to version 1 of the application and verify that it is working:

    [cloud_user@km]$ kubectl apply -f istio-1.0.6/samples/bookinfo/networking/virtual-service-all-v1.yaml

Update the virtual service file to point to version 2 of the service and verify that it is working. Edit istio-1.0.6/samples/bookinfo/networking/virtual-service-all-v1.yaml (using whatever text editor you like) and change this:

    - destination:
            host: reviews
            subset: v1

to this:

    - destination:
            host: reviews
            subset: v2

## Conclusion
We used Istio to deploy an application in a Kubernetes cluster, and made sure routing rules were sending traffic to whichever targets (versions of a backend service in this case) we wanted. We're done. Congratulations!