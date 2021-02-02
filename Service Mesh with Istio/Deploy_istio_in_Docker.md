# Istio in Docker
## The Scenario

Our organization has determined that we need to provide application versioning for our Docker-deployed application. We will need to install Istio and deploy the bookinfo application. Then we'll need to verify that it is loading only version 1 of the application (which has no review stars). Afterward, we'll need to route traffic to version 3 of the application (which has red stars).

## Log In
Log in to the environment using the credentials provided on the lab page, either in a terminal session on your local machine or by clicking Instant Terminal.

## Install Istio and Its Dependencies for Docker, Then Deploy the bookinfo Application
Install **docker-compose** and make it executable:

    [cloud_user@host]$ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-Linux-x86_64" -o /usr/local/bin/docker-compose  
    [cloud_user@host]$ sudo chmod +x /usr/local/bin/docker-compose

We'll run **docker-compose** real quick, just to make sure it works before we proceed. We'll see its help output if all is well.

Now we can download Istio and unpack it:

    [cloud_user@host]$ wget https://github.com/istio/istio/releases/download/1.0.6/istio-1.0.6-linux.tar.gz
    [cloud_user@host]$ tar -xvf istio-1.0.6-linux.tar.gz

Pre-configure kubectl for pilot:

    [cloud_user@host]$ kubectl config set-context istio --cluster=istio
    [cloud_user@host]$ kubectl config set-cluster istio --server=http://localhost:8080
    [cloud_user@host]$ kubectl config use-context istio

Create a **DOCKER_GATEWAY** environment variable:

    [cloud_user@host]$ export DOCKER_GATEWAY=172.28.0.1:

Bring up Istio's control plane:

    [cloud_user@host]$ docker-compose -f ./istio-1.0.6/install/consul/istio.yaml up -d

Remember that this may need to be repeated to ensure the pilot container starts.

Change **bookinfo.yaml**, and set port 30080 instead of port 9081:

    [cloud_user@host]$ sed -i 's/9081/30080/' ./istio-1.0.6/samples/bookinfo/platform/consul/bookinfo.yaml

Bring up the application:

    [cloud_user@host]$ docker-compose -f ./istio-1.0.6/samples/bookinfo/platform/consul/bookinfo.yaml up -d

Bring up the sidecars:

    [cloud_user@host]$ docker-compose -f ./istio-1.0.6/samples/bookinfo/platform/consul/bookinfo.sidecars.yaml up -d

Point a browser at the public IP address of the server, on the correct port: http://<PUBLIC_SERVER_IP>:30080/productpage.

## Route Application Traffic to reviews Version 1 and Confirm that Version 1 (with No Stars) is Loading

Route traffic to version 1

    [cloud_user@host]$ kubectl apply -f ./istio-1.0.6/samples/bookinfo/platform/consul/destination-rule-all.yaml
    [cloud_user@host]$ kubectl apply -f ./istio-1.0.6/samples/bookinfo/platform/consul/virtual-service-all-v1.yaml

If we refresh our web browser (the one we already had open looking at the application) we get no stars in the reviews section.

## Route the Traffic to Version 3 of the reviews Service (with Red Stars)

We need to modify the routing service subsets to read the labels for Version 3. Edit istio-1.0.6/samples/bookinfo/platform/consul/virtual-service-all-v1.yaml (using whatever text editor you like) and change this:

    - destination:
            host: reviews.service.consul
            subset: v1

to this:


    - destination:
            host: reviews.service.consul
            subset: v3
Then apply the changes with:

    [cloud_user@host]$ kubectl apply -f ./istio-1.0.6/samples/bookinfo/platform/consul/virtual-service-all-v1.yaml

Now if we refresh our web browser, we'll see red stars in the reviews section.

## Conclusion

We used Istio and Docker to deploy an application, then we made sure routing rules were sending traffic to whichever targets (versions of a backend service in this case) we wanted. We're done. Congratulations!