# PROBELM-SOLVING-NOTES-RECORD

## PROMBLEM 1

  I was unable to access the cluster.

## Solution

## PROBLEM 2

I was unable to push the docker images from my local system ko GCP GCR..

## Solution
First thing first set project
  
    gcloud config set project <project-id>
    gcloud auth login

Check docker is installed on your system or not. If not then install it with below command

    sudo apt install docker.io

now add your local user in docker group. so your user can have the access of docker group

    sudo usermod -aG docker $(whoami) or sudo usermod -aG docker <your-user-name>
    
now refresh docker group for reflecting effect

    newgrp docker       

verify your user in docker group

    grep docker /etc/group

now add authentication for docker. In gcp it is require. after this you can have access of GCP GCR for pushing images..
   
    gcloud auth configure-docker 
    or 
    gcloud auth configure-docker gcr.io

Ensure that your Google Cloud IAM permissions are correct for your gcp user. Verify permission you need to have **owner** or **roles/artifactregistry.writer** role.

You can check the permissions with the following command:
  
    gcloud projects get-iam-policy <project-id>

    check docker service status

    sudo systemctl status docker

Make it start if it is not running

    sudo systemctl start docker

TAG-IMAGE AND PUSH IMAGE:

     docker tag lunch-benefits-services:latest gcr.io/pcpeprod/new-repository:latest
     docker push gcr.io/pcpeprod/new-repository:latest


Check Docker Configuration
Ensure that Docker is configured to use gcloud for authentication:

    cat ~/.docker/config.json

  You should see entries similar to:

    {
      "credHelpers": {
        "gcr.io": "gcloud",
        "us.gcr.io": "gcloud",
        "eu.gcr.io": "gcloud",
        "asia.gcr.io": "gcloud",
        "staging-k8s.gcr.io": "gcloud",
        "marketplace.gcr.io": "gcloud"
      }
    }

Verify Your Authentication

Ensure you're properly authenticated:

    gcloud auth print-access-token

    if you can get token, it's mean your docker are able to communicate with GCP GCR.

You can also try using this token to log in manually:

    gcloud auth print-access-token | docker login -u oauth2accesstoken --password-stdin https://gcr.io

## Mistake

I had added my user in docker group it mean with normal user i can push docker images 

like:

    docker push <image-name>

but i was mistakenly using **sudo** with docker push command. it mean docker is now trying to use root user for pushing image on gcr.. but root user is not add in docker group..

**so once i removed sudo with docker push command. I can successfully push image to GCP GCR. because docker now uses normal user that are also a part of docker group** 

# PROBLEM 3

I had the access of jenkins vm. But because of OS re-installation i lost it from my terminal. 
Solution:
  
Because i knew the jenkins vm username and password.. I have the below command to get vm access again.

    ssh hasasn@jenkins-vm ip or dns address
or it ask you to give password. After giving password you can get access of your vm..
  
    ssh-copy-id hassan@jenkins-vm ip or dns address         
it will copying the key to the remote server..

with this i got the access. or because i had set password against my username. so i can simplily
  
# PROBLEM 4 Request not passed through kubernetes service to deployment..

    Solution:
    
      Basically service was not being passed through traffic to deployment. **It was a port miss-match issue. My code was running on port 8000. I have confirmed this by veiwing main.py. And my container port expose port 5000. I have confirm this by veiwing Dockerfile. And I have also written containerport and targetport my kubernetes deployment and service templates was 5000. it was a wrong approach.
    
    **Always remember to set the port for container to the port on which code is running** mean if code is running on 8000 then the container port should also be set on 8000.
    
    what to do this in docker and kubernetes...
    
    **kubernetes** if code is running on 8000(for this see code file like main.py for python) then in dockerfile expose port for container should set on same 8000 port regardless whatever is set in entrypoint(set same code pod here.) and in kubernetes deployment template containerport(it is the port for the container) is also set to same 8000 and in kubernetes service template targetport is also set to 8000...
    
    Traffic flow: from fronted to backend kubernetes serivce(loadbalancer) on toport(for most case we set port 80) to targetport of container on 8000  to code running on same 8000 port..
    
    **troubleshoot**: 
    
      1- if service not send traffic to deployment and you want to see if the code is running or not, then you can see my port-forwarding the **pod** on port on which the code is running(suppose code is running on 8000 then you should port forward the pod to 8000). and try and see the code respose **locally** from **browser** or from terminal with **curl**  with port on which code is running... you can also check it from the postman.
    
      like port forwarding command:   **kubectl port-forward pod/pod-name anyport(like 7171):codeport(like 8000)**
      go to browser and see the response: **http://localhost:7171** and for terminal use **curl http://localhost:7171** 
    
      with this you can confirm that code is running on pod successfully. you can also run the same code on local system by creating docker container... same see response locally..
    
      2- you can expose the service(mean make the service public) by changing the annotation..
    
      for this edit the deployment service and change annotation from Internal to External. It will go you the public ip in 2min, use this on local browser to see the respose.**(http://publicip of service)** if service and deployment configured correctly it will definatily give you the respose on browser. otherwise it will make you confirm that there is port or label mismatch b/w service and deployment.. 
    
      mean while you can see the continous logs with: **kubectl logs pod/pod-name -f**  
      
    **Docker** if code is running on 8000(for this see code file like main.py for python) then in dockerfile expose port for container should set on same 8000 port regardless whatever is set in entrypoint(set same code pod here.) and for starting the container you can map any external port to same internal container port during running the container using the docker run command...
    
      docker run --port externalport:internetport(containerport, same on which code is running) 
    
    
    Most clearn version:
    -------------------
    
    **Incident Report: Port Mismatch Causing Traffic Issue to Deployment**
          
        Incident Summary: The service wasn't passing traffic to the deployment due to a port mismatch issue. Upon investigation, I found that the code was running on port 8000 (confirmed by checking main.py), but the container was exposing port 5000 (confirmed by reviewing the Dockerfile). In the Kubernetes deployment and service templates, both the containerPort and targetPort were set to 5000, which was incorrect.
          
          Key Lesson: Always ensure that the port exposed by the container matches the port on which the code is running. If the code is running on port 8000, the container should expose port 8000, and the Kubernetes deployment and service should also set the containerPort and targetPort to 8000.
          
    Correct Approach in Docker and Kubernetes
          
    1. Kubernetes Configuration
    
          If the code runs on port 8000 (you can check this by looking at the code, e.g., main.py for Python), then:
       
    Dockerfile: Ensure the Dockerfile exposes the correct port for the container:
          
          # Expose the port on which the code is running (e.g., 8000)
          EXPOSE 8000
          
    Kubernetes Deployment Template: Set the containerPort in the deployment to match the port on which the code is running:
          
          containers:
          - name: your-container-name
            image: your-image-name
            ports:
            - containerPort: 8000  # This should match the code port
          
    Kubernetes Service Template: Set the targetPort to the same port on which the container is running (in this case, 8000). The port (typically 80 for HTTP traffic) forwards traffic to the container's targetPort.
          
          spec:
            ports:
            - port: 80          # Frontend service port (commonly set to 80 for HTTP traffic)
              targetPort: 8000   # Container's port, which matches the code port
          
    Traffic Flow Explanation:
          
          Traffic is routed from the frontend (e.g., LoadBalancer) to the Kubernetes service on port 80.
          The service forwards traffic to the deployment's targetPort (8000).
          Inside the deployment, the container listens on port 8000, where the code is running.
          Troubleshooting Steps
          
    Check if Code is Running:
          
          If the service isn’t sending traffic to the deployment, verify if the code is running by port-forwarding to the pod on the port where the code runs (e.g., 8000).
          Example command to forward pod traffic to your local machine:
          
    
          kubectl port-forward pod/<pod-name> 7171:8000
         
     Then check the response locally in your browser:
    
          http://localhost:7171
          
          Or use curl from the terminal:
          
          
          curl http://localhost:7171
          This helps confirm if the code inside the pod is working correctly.
          
          Expose the Service Publicly:
          
          If you need to make the service public, change the annotation in your Kubernetes service from Internal to External. This will assign a public IP, which you can access in the browser:
          
         
          http://<public-ip-of-service>
          If the service is correctly configured, it should respond. Otherwise, it will confirm there’s a port or label mismatch between the service and deployment.
          
          Check Logs: You can continuously check the logs to troubleshoot further:
          
         
          kubectl logs pod/<pod-name> -f
          Correct Approach for Docker
          Dockerfile: Ensure the exposed port in the Dockerfile matches the port the code is running on (e.g., 8000):
          
    dockerfile
         
          EXPOSE 8000
          Docker Run Command: When starting a container, map any external port to the internal container port where the code is running.
          
          Example command:
          
    
          docker run --port 7171:8000
          This will map port 7171 on your local machine to port 8000 inside the container, allowing you to access the running code locally on http://localhost:7171.
          
    Conclusion
    
          Always ensure the container port and the code port are aligned throughout the setup in both Docker and Kubernetes. Misconfigured ports will cause traffic to not reach your services properly.

## scp command to send file to remote vm 

  scp  cors.lua Dockerfile haproxy.cfg hassan@34.45.113.234:/home/hassan

## I was unable to establish communication b/w 2 contianer.. 

  followed this steps...

  When running both HAProxy and the backend service in separate containers, referencing localhost in HAProxy's configuration won't work as expected. This is because localhost refers to the container itself, not to the host machine or other containers. In Docker, containers have isolated networks, so you need to connect them properly for HAProxy to communicate with the backend service.

    Here’s how you can resolve the issue:
    
    1. Use Docker Networking
    Docker containers can communicate with each other via network names, so you need to make sure both HAProxy and your backend service are on the same Docker network.
    
    Step 1: Create a Docker network
    
    docker network create my_network
    
    Step 2: Run your backend service on this network
    For example, if your backend service is an HTTP server running on port 8000:
    
    
    docker run --name backend_service --network my_network -p 8000:8000 nginx
    
    Step 3: Run HAProxy on the same network
    Now, run your HAProxy container on the same network:
    
    bash
    Copy code
    docker run --name haproxy1 --network my_network -p 80:80 haproxy:v1
    2. Update HAProxy Configuration
    In your haproxy.cfg, instead of using localhost, use the backend service’s container name (backend_service in this example). Docker's internal DNS will resolve the container name to its IP address.
    
    Change this line in your backend web_backend section under haproxy.cfg file
    
    server local_server <backend container name or ip>:<backend contianer internel port> check
    server local_server backend_service:80 check
    
## haproxy backend configuration tips.. or communication from localhost to docker container and from docker container to other docker contianer..

      Accessing Containers with HAProxy

      When dealing with Docker containers, understanding how to route traffic between them is crucial, especially when using a reverse proxy like HAProxy.
      
      1. Accessing a Container from the Host
      If your container is running on your local machine (localhost) and is mapped to an external port (e.g., port 8000) to internal port80, you can access it directly from your host machine using curl. For example:
      
      curl http://localhost:8000
      
      In this case, port 8000 maps to port 80 inside the container, allowing you to receive responses from the service running inside it.
      
      HAProxy Configuration:
      
      You can configure HAProxy to point to localhost:8000 in the backend section because HAProxy runs on the same host and can forward requests to the container:
      
      haproxy
      Copy code
      backend my_backend
          server backend_server localhost:8000
     
       2. Running HAProxy Inside a Container
      When you run HAProxy inside a container, it needs to communicate with another container (like NGINX) for routing requests. To achieve this, you must set up a Docker network and attach both containers to it. This enables them to communicate privately.
      
      Creating a Docker Network:
      
      To create a network, use the following command:
    
      docker network create my_network
      
      Then, when running your containers, use the --network flag to attach them to the created network:

      docker run --network my_network --name my_nginx nginx
      docker run --network my_network --name my_haproxy haproxy:v1
     
      3. Configuring HAProxy to Communicate with NGINX
      Since both containers are now on the same network, they can communicate using their container names or IP addresses. In the HAProxy configuration, specify the backend as follows:
      
      haproxy
      Copy code
      backend my_backend
          server backend_server my_nginx:80    ----> <backendcontainer-name/ip>:<internalport>
      This way, HAProxy can reach the NGINX container through its name (my_nginx) and port 80.
      
      4. Verifying Communication
      To verify that HAProxy can reach the NGINX container:
      
      Enter the HAProxy Container:

      docker exec -it my_haproxy /bin/sh  # or /bin/bash depending on your image
      
      Use curl to Test Connectivity: Inside the HAProxy container, run:

      curl http://my_nginx:80           ----> install curl if not available inside the container... -->apt-get -y update; apt-get -y install curl
      
      This should return the response from the NGINX server.
      
      Conclusion
      By setting up your containers on the same Docker network, you enable seamless communication between them. This configuration allows HAProxy to effectively route traffic to your backend services using their internal container names and ports.
      
      This revised explanation is designed to be clearer and more informative while guiding you through the process step-by-step. Let me know if you need any further modifications or details!
      
    for more detail
    ---------------  
      
      You said:
      backend my_backend
          server backend_server my_nginx:80
      do i need to give the internal port if haproxy is running inside the container
      ChatGPT said:
      ChatGPT
      Yes, when configuring HAProxy to communicate with your backend service (in this case, NGINX) from within a container, you should specify the internal port of the backend service.
      
      r.
      
      Example Configuration
      In your HAProxy configuration, you would typically write:
      
      haproxy
      Copy code
      backend my_backend
          server backend_server my_nginx:80
      Explanation:
      my_nginx: This is the hostname of the NGINX container (as long as both containers are in the same Docker network).
      80: This is the internal port where NGINX is listening. Since HAProxy is communicating with NGINX inside the Docker network, you must specify this internal port.
      Summary
      So, yes, you need to specify the internal port (80 in this case) in your HAProxy backend configuration to ensure that HAProxy can correctly route the requests to the NGINX service running in another container.
      
## Dockerfile understanding

    When working locally, such as setting up HAProxy, you need to package it on your local system. This means necessary files and configurations will be placed in directories like /etc/haproxy after the installation of the HAProxy package.
    
    However, if you want to perform the same setup within a Docker container using a Dockerfile, you can specify a base image in the FROM directive. For example, by defining FROM haproxy:2.4-bookworm, you leverage the base image, which automatically provides the environment and necessary files without needing to install them manually like you do in your local system.
    
    In addition, you can install any other required packages using the RUN command in the Dockerfile. This allows you to customize the container's environment as needed, all while ensuring that the base image takes care of the core configurations and dependencies.
      
      
      
      
      
      
      
