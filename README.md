# PROBELM-SOLVING-NOTES-RECORD

#### PROMBLEM 1

  I was unable to access the cluster.

#### Solution

#### PROBLEM 2

    I was unable to push the docker images from my local system ko GCP GCR..

    Solution
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
    
#### Mistake

    I had added my user in docker group it mean with normal user i can push docker images 
    
    like:
    
        docker push <image-name>
    
    but i was mistakenly using **sudo** with docker push command. it mean docker is now trying to use root user for pushing image on gcr.. but root user is not add in docker group..
    
    **so once i removed sudo with docker push command. I can successfully push image to GCP GCR. because docker now uses normal user that are also a part of docker group** 
    
#### PROBLEM 3

    I had the access of jenkins vm. But because of OS re-installation i lost it from my terminal. 
    Solution:
      
    Because i knew the jenkins vm username and password.. I have the below command to get vm access again.
    
        ssh hasasn@jenkins-vm ip or dns address
    or it ask you to give password. After giving password you can get access of your vm..
      
        ssh-copy-id hassan@jenkins-vm ip or dns address         
    it will copying the key to the remote server..
    
    with this i got the access. or because i had set password against my username. so i can simplily
      
#### PROBLEM 4: Request not passed through kubernetes service to deployment..

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

#### Scp command to send file to remote vm 

  scp  cors.lua Dockerfile haproxy.cfg hassan@34.45.113.234:/home/hassan

#### I was unable to establish communication b/w 2 contianer.. 

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
    
#### Haproxy backend configuration tips.. or communication from localhost to docker container and from docker container to other docker contianer..

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
      
#### Dockerfile understanding

    When working locally, such as setting up HAProxy, you need to package it on your local system. This means necessary files and configurations will be placed in directories like /etc/haproxy after the installation of the HAProxy package.
    
    However, if you want to perform the same setup within a Docker container using a Dockerfile, you can specify a base image in the FROM directive. For example, by defining FROM haproxy:2.4-bookworm, you leverage the base image, which automatically provides the environment and necessary files without needing to install them manually like you do in your local system.
    
    In addition, you can install any other required packages using the RUN command in the Dockerfile. This allows you to customize the container's environment as needed, all while ensuring that the base image takes care of the core configurations and dependencies.

## Jenkins job catagory

  We can categorize jobs in Jenkins by using Views. Views allow us to organize and group related jobs, making it easier to manage and access them.

#### Got problem during envoy install using helm

  Problem:

  helm install envoy ./envoy/
  
  Error: INSTALLATION FAILED: unable to build kubernetes objects from release manifest: resource mapping not found for name: "envoy" namespace: "" from "": no matches for kind "PodDisruptionBudget" in version "policy/v1beta1"
  ensure CRDs are installed first


  solution:

      The error you are encountering, specifically related to PodDisruptionBudget and policy/v1beta1, happens because the PodDisruptionBudget API version has been updated in Kubernetes. Starting from Kubernetes version 1.21, PodDisruptionBudget has moved from policy/v1beta1 to policy/v1.

    Go to the template > PodDisruptionBudget.yaml and change api version from policy/v1beta1 to policy/v1.

    new template:

    apiVersion: policy/v1
    kind: PodDisruptionBudget
    metadata:
      name: envoy
    spec:
      minAvailable: 1
      selector:
        matchLabels:
          app: envoy


    
    or 

    Most likely the problem is not related to missing CRDs but to the kubernetes version. I assume you are using the latest K3S version, which is v1.25.4. **PodDisruptionBudget was moved from policy/v1beta1 to policy/v1 in version v1.25.** As the Envoy helm chart that you are using does not seem to be actively maintained, probably you will have to downgrade K3S or find a different chart.


#### Calling from one service to another service with FQDN in the same namespace..

    For this we can use **<service-name>.<namespace>.svc.cluster.local**

    in my case it would be come **nginx-service.default.svc.cluster.local**

    or you can also give service name simply... to the service which need to communicate to the other service... like in my case mny envoy proxy service need to communicate nginx service running as envoy proxy backend... so i can give nginx service name or FQDN to envoy proxy. so envoy proxy can communicate with ngnix service...


    With FQDN
    ---------
    
    clusters:
          - name: service_google
            connect_timeout: 0.25s
            type: LOGICAL_DNS
            dns_lookup_family: V4_ONLY
            lb_policy: ROUND_ROBIN
            hosts:
              - socket_address:
                  address: nginx-service.default.svc.cluster.local
                  port_value: 80 

    OR WITH NAME
    ------------ 
        
    clusters:
          - name: service_google
            connect_timeout: 0.25s
            type: LOGICAL_DNS
            dns_lookup_family: V4_ONLY
            lb_policy: ROUND_ROBIN
            hosts:
              - socket_address:
                  address: nginx-service
                  port_value: 80

#### Kubectl port forwarding command format

  kubectl port-forward <resource_type>/<resource_name> <local_port>:<remote_port>

#### Cluster node version upgration

  -  First drain the node with gcloud command, by doing this cluster will not send any traffic to the draining node... and will move node resources to master node. and make the node **cordan**.
  -  After this create a new node pull having latest node version. Once done uncordan the node...                

    these are the process for node upgradion. refer to the documentation before using these steps.

### ingress issues

    - GCP will take 10 to 15 mins for properly setting up the ingress. 
    - Once ingress become ready. test it from local console.. like **curl http://envoy.disearch.ai/**
    - Once get result. Now apply GCP managed certificate with ingress. First time it will take 20 to 30 mint to issue the certificate.. and attaching with ingress loadbalancer. for checking status go to gcp certificate manager and see certicate status. (Provioning: means it is provisioning, ACTIVE: mean certificate is active). **Here IN USE** section will tell you certificate is attached with ingress.
    - Go to ingress and see ingress will route specfic domain to backend service. and service will sent it to the deployment. If it is ingress is showing that **backend service is UNHEALTHY**, then check your backend service and pods..  Hit traffic on backend service pod directly and see the response(for this you need to go to other pod and hit traffic from other pod to backendservice pod and see response.). and you can also go inside the backend service pod and hit from inside the same pod, like ** curl http://localhost:<code listening port>**
  - if any thing not work, you simply need to recreate ingress again... just delete the ingress from kubernetes and redeploy the ingress manifest again...
  - once done every thing. test ingress domain from local system like **curl -kv https://envoy.disearch.ai/** and see the response... **-kv** will ignore the certificate..
  
### Face same GCP ingress issue again.. SOME BACKEND SERVICE ARE UNHEATHLY

  Solution:
  - I want to backend service and send traffic from other pod to the backend service.. (like: if go inside the other pod and then send traffic(ping backend pod) to see the backend pod response like **curl http://<backendservice-name>:<backendservice-port>**). And i have found the same tls handshake error. I remove the below line in my envoy.yaml file and redeploy the deployment using helm. And it started working..

      transport_socket:
                name: envoy.transport_sockets.tls
                typed_config:
                  "@type": type.googleapis.com/envoy.extensions.transport_sockets.tls.v3.UpstreamTlsContext
                  sni: envoy.disearch.ai   

    And GCR ingress also showing HEALTHY Status


## Command to manually test the certificate

   curl -vI https://envoy.disearch.ai or curl -kv https://envoy.disearch.ai --> -kv ingnore the certificate
    
   *   Trying 34.117.212.151:443...
   * TCP_NODELAY set
   * Connected to envoy.disearch.ai (34.117.212.151) port 443 (#0)
   * ALPN, offering h2
   * ALPN, offering http/1.1
   * successfully set certificate verify locations:
   *   CAfile: /etc/ssl/certs/ca-certificates.crt
     CApath: /etc/ssl/certs
   * TLSv1.3 (OUT), TLS handshake, Client hello (1):
   * TLSv1.3 (IN), TLS handshake, Server hello (2):
   * TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
   * TLSv1.3 (IN), TLS handshake, Certificate (11):
   * TLSv1.3 (IN), TLS handshake, CERT verify (15):
   * TLSv1.3 (IN), TLS handshake, Finished (20):
   * TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
   * TLSv1.3 (OUT), TLS handshake, Finished (20):
   * SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
   * ALPN, server accepted to use h2
   * Server certificate:
   *  subject: CN=envoy.disearch.ai
   *  start date: Oct  8 17:25:10 2024 GMT
   *  expire date: Jan  6 18:16:42 2025 GMT
   *  subjectAltName: host "envoy.disearch.ai" matched cert's "envoy.disearch.ai"
   *  issuer: C=US; O=Google Trust Services; CN=WR3
   *  SSL certificate verify ok.
   * Using HTTP2, server supports multi-use
   * Connection state changed (HTTP/2 confirmed)
   * Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
   * Using Stream ID: 1 (easy handle 0x558f5d3a5340)
   > HEAD / HTTP/2
   > Host: envoy.disearch.ai
   > user-agent: curl/7.68.0
   > accept: */*
   > 
   * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
   * TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
   * old SSL session ID is stale, removing
   * Connection state changed (MAX_CONCURRENT_STREAMS == 100)!
   < HTTP/2 200 
   HTTP/2 200 
   < server: envoy
   server: envoy
   < date: Tue, 08 Oct 2024 21:27:50 GMT
   date: Tue, 08 Oct 2024 21:27:50 GMT
   < content-type: text/html
   content-type: text/html
   < content-length: 615
   content-length: 615
   < last-modified: Wed, 02 Oct 2024 15:13:19 GMT
   last-modified: Wed, 02 Oct 2024 15:13:19 GMT
   < etag: "66fd630f-267"
   etag: "66fd630f-267"
   < accept-ranges: bytes
   accept-ranges: bytes
   < x-envoy-upstream-service-time: 0
   x-envoy-upstream-service-time: 0
   < via: 1.1 google
   via: 1.1 google
   < alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
   alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
   
   <
   * Connection #0 to host envoy.disearch.ai left intact  

## TESTING
  
  Command to test envoy rate limit. GO inside any gke pod and use this.. it will send traffic to your service running envoy. and you can test the envoy rate limit feature

    while true; do curl http://envoy-service:10000/headers; done

  Bash script to test rate limiting

    # Initialize counters
    count_200=0
    count_429=0
    count_503=0
    count_502=0
    count_other=0
    
    # Loop to send 60 requests
    for i in {1..60}; do
      echo "Request $i:"
      
      # Send request and capture the HTTP response code
      response=$(curl -s -o /dev/null -w "%{http_code}" -I -X GET https://envoy.disearch.ai/)
    
      # Display the response code for each request
      echo "Response code: $response"
      
      # Increment counters based on response code
      if [ "$response" -eq 200 ]; then
        ((count_200++))
      elif [ "$response" -eq 429 ]; then
        ((count_429++))
      elif [ "$response" -eq 503 ]; then
        ((count_503++))
      elif [ "$response" -eq 502 ]; then
        ((count_502++))
      else
        ((count_other++))
      fi
      
      echo ""  # Add a blank line between results for readability
    done
    
    # Print the counts at the end
    echo "Summary:"
    echo "200 responses: $count_200"
    echo "429 responses: $count_429"
    echo "503 responses: $count_503"
    echo "502 responses: $count_502"
    echo "Other responses: $count_other"

## Command to Test CORS

To test your CORS configuration, you can use the following curl commands:
            
Testing CORS with origin http://test-origin-1.com:
            
      curl -i -X OPTIONS https://envoy.disearch.ai \
      -H "Origin: http://test-origin-1.com" \
      -H "Access-Control-Request-Method: POST" \
      -H "Access-Control-Request-Headers: content-type"

Expected CORS Response for Origin http://test-origin-1.com:
      
            HTTP/2 200 
            access-control-allow-origin: http://test-origin-1.com
            access-control-allow-methods: POST
            access-control-allow-headers: content-type
            access-control-max-age: 100
            date: Wed, 09 Oct 2024 18:28:46 GMT
            server: envoy
            content-length: 0
            via: 1.1 google
            alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000

Testing CORS with origin http://test-host-2.com:
            
            curl -i -X OPTIONS https://envoy.disearch.ai \
            -H "Origin: http://test-host-2.com" \
            -H "Access-Control-Request-Method: POST" \
            -H "Access-Control-Request-Headers: content-type"
            
Expected CORS Response for Origin http://test-host-2.com:
            
            HTTP/2 200 
            access-control-allow-origin: http://test-host-2.com
            access-control-allow-methods: POST
            access-control-allow-headers: content-type
            access-control-max-age: 100
            date: Wed, 09 Oct 2024 18:34:32 GMT
            server: envoy
            content-length: 0
            via: 1.1 google
            alt-svc: h3=":443"; ma=2592000,h3-29=":443"; ma=2592000
            
Explanation of CORS Configuration

            In your Envoy CORS configuration, you have specified the following:
            Origins:
            
            http://test-origin-1.com
            http://test-host-2.com
            
These origins are explicitly allowed in the allow_origin_string_match section, meaning the server will accept CORS requests coming from either of these two domains.
            
Allowed Methods:
            
            POST requests are allowed. This is defined in the allow_methods section.
            
Allowed Headers:
            
            The header content-type is permitted, as specified in the allow_headers section.
            
Max Age:
            
            The preflight response can be cached for 100 seconds, defined by max_age: "100". This reduces the number of preflight requests needed for the same method and headers from the same origin.
            
Relevant Envoy Configuration Section
            
            Here is the relevant section of your Envoy configuration where the CORS policy is applied:
            
            virtual_hosts:
              - name: local_service
                domains: ["*"]
                typed_per_filter_config:
                  envoy.filters.http.cors:
                    "@type": type.googleapis.com/envoy.extensions.filters.http.cors.v3.CorsPolicy
                    allow_origin_string_match:
                    - safe_regex:
                        regex: ".*"
                    allow_headers: "content-type,x-grpc-web"
                    allow_methods: "GET, POST, OPTIONS"
                    expose_headers: "custom-header-1,custom-header-2"
                    max_age: "3600"
                    allow_credentials: true
                routes:
                  - match:
                      prefix: "/"
                    route:
                      host_rewrite_literal: envoy.disearch.ai
                      cluster: frontend_service
                    typed_per_filter_config:
                      envoy.filters.http.cors:
                        "@type": type.googleapis.com/envoy.extensions.filters.http.cors.v3.CorsPolicy
                        allow_origin_string_match:
                        - exact: "http://test-origin-1.com"
                        - exact: "http://test-host-2.com"
                        allow_headers: "content-type"
                        allow_methods: "POST"
                        max_age: "100"
            
In this configuration, you have specified both global and route-specific CORS policies. The allow_origin_string_match directive ensures that only the predefined origins are permitted to make cross-origin requests. You also control allowed methods, headers, and the cache duration for preflight requests (via max_age).
            
            
Conclusion
            
            By using the curl commands and configuring your CORS policy as described, Envoy is correctly handling CORS requests from the specified origins (http://test-origin-1.com and http://test-host-2.com). Ensure that any additional requests from other origins or with different methods/headers align with your configuration.


   

## Created cronjob to purage image

  azeer created cronjob to purage image on jenkins.

## What is the difference b/w deployment and statefulset.

  The key difference b/w deployment and statefulset is "statefulset alway maintain its state". Whenever the pod is recreated by deployment or replicaset, it gains a new identifier(name), mean kubernetes would generate a new hash code  of a new pod that is recreated. but in statefullset it does not happen every recreated pod comes with the same name.

  Detail:  StatefulSet can maintain the identity of its Pods, becoming an indispensable component for applications where you want to maintain the state. As we have seen countless times. Each time a Pod is recreated by a Deployment or ReplicaSet resource, it gains a new identifier, which means it gains a new name.Kubernetes generates a new hash code to compose each name of a new Pod that is recreated.

  In the case of StatefulSets, this does not happen, every time a Pod is recreated, it comes with the same name.

  **With StatefulSets, you will take advantage of the following features.**

  **Unique network identifiers.**

    This means that if your Pod goes down or is restarted for any reason, **when it comes back up, it can still be accessed by the same DNS address**, i.e. the same name.

  **Persistent storage.**

    This means that your Pods will have a PersistentVolume even after deleting the Pods, and even after deleting the StatefulSet, your data will not be lost. When you delete a StatefulSet, all Pods associated with it, that is, all those Pods that it is managing will be automatically removed, but the volumes will persist. That is, they will not be deleted and your data will continue to live.

  **Another fundamental characteristic of StatefulSets resources is that implementation, scaling and rolling updates are carried out orderly,**

   respecting a numerical order for these operations. You can know exactly the sequence in which your Pods will be deployed, scaled, and even the sequence in which they will be removed during the scale-down operation. Using Deployments or ReplicaSets.

  You don't have these characteristics, but with StatefulSets, we have unique identifiers in the Pods,
  
  so we can know which one will undergo a modification in these cases.

## Roll undo command for kubernetes deployment
  
    kubectl rollout history deployment vertexai-citation-deployment
    kubectl rollout undo deployment vertexai-citation-deployment
    kubectl rollout history deployment vertexai-citation-deployment

## Edit kubernetes resources

  Remember: 
  Always edit the resource deployment or StatefulSet, not the pod directly. If you edit the pod directly instead of editing the deployment or StatefulSet, when the pod restarts, its corresponding StatefulSet or deployment will recreate the pod with the previous configuration, and any new changes made to the pod will be lost. Therefore, it's best practice to edit the deployment or StatefulSet instead of the pod. This way, the pod will receive the updated changes from the deployment or StatefulSet with every restart.
  
  You can delete pods to pull the latest images, provided the pod and the deployment use the same image tags. The pod will automatically pull the latest image. However, if you need to update the image tag, you should update the deployment, not the pod. Otherwise, you may face the issue mentioned above, where changes are overwritten.
      
  
## I encountered an issue while trying to build and push an image from my local system to Google Cloud Artifact Registry.

    I encountered an issue while trying to build and push an image from my local system to Google Cloud Artifact Registry using the command:
    
      gcloud builds submit
    
    The build process succeeded, but when it attempted to push the image, I received the following error:
    
    denied: Permission "artifactregistry.repositories.uploadArtifacts" denied on resource "projects/disearchmt-dev/locations/us/repositories/gcr.io" (or it may not exist)
    ERROR: failed to push because we ran out of retries.
    ERROR: error pushing image "gcr.io/disearchmt-dev/image-pubsub": generic::unknown: retry budget exhausted (10 attempts): step exited with non-zero status: 1
    ERROR: (gcloud.builds.submit) build <build-id> completed with status "FAILURE"
    
    To resolve this, I granted Artifact Registry Writer and Storage Admin permissions to the user account (mursleen) and asked them to try the gcloud builds submit command again. However, the issue persisted. I then suggested using Docker commands instead, specifically running:
    
      gcloud auth configure-docker
    
    After configuring Docker authentication, the user was able to successfully push the image to the Artifact Registry. However, when deploying this image to a Cloud Run service, we encountered an error stating that the container was not running on port 8080. The user then requested I investigate the original gcloud builds submit command error, as it had previously worked fine for deploying Cloud Run services.
    
    Upon troubleshooting, I found the following insight online:
    
    “I don’t use Docker commands to push the image; I let Cloud Build recognize the language and build the image with the command: gcloud builds submit --pack image=europe-west3-docker.pkg.dev/...... I encountered the same error due to insufficient permissions for the Cloud Build service account on the Artifact Registry. The solution was to assign the ‘Artifact Registry Writer’ role to the Cloud Build service account.”
    
    After reading this, I understood that the issue was related to Cloud Build using a service account without the correct permissions for pushing images to the Artifact Registry. I verified that the service account used by Cloud Build in the disearchmt-dev project did not exist in IAM.(for verify this by going to cloud build>setting  and have seen that the service account attached to cloud build have not assign the permission to communicate to gcr artifact) I then created a comparable service account with the Artifact Registry Writer role and assigned it as the default service account for Cloud Build in my project.
    
    After these changes, I asked the developer to retry the gcloud builds submit command, and this time, the image was built and pushed successfully. The image was also used in Cloud Run without any issues. Adding the Artifact Registry Writer permission resolved the problem.


## ETCD BACKUP
    
    (Optional) Additional information about ETCDCTL UtilityETCDCTL is the CLI tool used to interact with ETCD.ETCDCTL can interact with ETCD Server using 2 API versions – Version 2 and Version 3. By default it’s set to use Version 2. Each version has different sets of commands.
    
    For example, ETCDCTL version 2 supports the following commands:
    
    etcdctl backup
    etcdctl cluster-health
    etcdctl mk
    etcdctl mkdir
    etcdctl set
    Whereas the commands are different in version 3
    
    etcdctl snapshot save
    etcdctl endpoint health
    etcdctl get
    etcdctl put
    To set the right version of API set the environment variable ETCDCTL_API command
    
    export ETCDCTL_API=3
    
    When the API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don’t work. When API version is set to version 3, version 2 commands listed above don’t work.
    
    Apart from that, you must also specify the path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path. We discuss more about certificates in the security section of this course. So don’t worry if this looks complex:
    
    
    --cacert /etc/kubernetes/pki/etcd/ca.crt
    --cert /etc/kubernetes/pki/etcd/server.crt
    --key /etc/kubernetes/pki/etcd/server.key
    So for the commands, I showed in the previous video to work you must specify the ETCDCTL API version and path to certificate files. Below is the final form:
    
    
    kubectl exec etcd-controlplane -n kube-system -- sh -c "ETCDCTL_API=3 etcdctl get / --prefix --keys-only --limit=10 --cacert /etc/kubernetes/pki/etcd/ca.crt --cert /etc/kubernetes/pki/etcd/server.crt --key /etc/kubernetes/pki/etcd/server.key"
    
    
    
    
    
    
    ***********************************************
    
    WORKING WITH ETCDCTL
    
    etcdctl is a command line client for etcd.
    
    In all our Kubernetes Hands-on labs, the ETCD key-value database is deployed as a static pod on the master. The version used is v3.
    
    To make use of etcdctl for tasks such as back up and restore, make sure that you set the ETCDCTL_API to 3.
    
    You can do this by exporting the variable ETCDCTL_API prior to using the etcdctl client. This can be done as follows:
    
    export ETCDCTL_API=3
    
    On the Master Node:
    
    Image
    
    To see all the options for a specific sub-command, make use of the -h or –help flag.
    
    For example, if you want to take a snapshot of etcd, use:
    
    etcdctl snapshot save -h and keep a note of the mandatory global options.
    
    Since our ETCD database is TLS-Enabled, the following options are mandatory:
    
    –cacert               verify certificates of TLS-enabled secure servers using this CA bundle
    
    –cert                  identify secure client using this TLS certificate file
    
    –endpoints=[127.0.0.1:2379] This is the default as ETCD is running on master node and exposed on localhost 2379.
    
    –key                 identify secure client using this TLS key file
    
    For a detailed explanation on how to make use of the etcdctl command line tool and work with the -h flags, check out the solution video for the Backup and Restore Lab.
    
    
    
    ***********************************************
    command to give etcd backup
    
    ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379
     \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save /opt/snapshot-pre-boot.db
    

## Command to stop, remove all container and images..

  sudo docker stop $(sudo docker ps -aq)
  sudo docker rm $(sudo docker ps -aq)
  sudo docker rmi -f $(sudo docker images -q)

## Run bash script and get ouput in .txt file..

  sudo bash docker-bench-security.sh > cis_benchmark_output.txt

## jenkins pipline for set images in kubernetes
  
  gcloud config set project disearch
  docker build -f Dockerfile -t gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT .
  docker tag gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT gcr.io/disearch/multi-model-doc-chat:latest
  docker push gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT
  docker push gcr.io/disearch/multi-model-doc-chat:latest
  gcloud container clusters get-credentials search-app-dev --zone us-central1-c --project disearch
  kubectl set image deployment/multi-model-doc-chat-deployment multi-model-doc-chat=gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT
  
  gcloud config set project disearchmt-dev
  docker tag gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT gcr.io/disearchmt-dev/multi-model-doc-chat:dev-$GIT_COMMIT
  docker push gcr.io/disearchmt-dev/multi-model-doc-chat:dev-$GIT_COMMIT
  gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project disearchmt-dev
  kubectl set image deployment/multi-model-doc-chat-deployment multi-model-doc-chat=gcr.io/disearchmt-dev/multi-model-doc-chat:dev-$GIT_COMMIT
    

## Argo CD supports managing multiple clusters from a single Argo CD instance, so you can use a single Argo CD setup to deploy applications across multiple GCP GKE clusters. Here’s how it can work:

      Single Argo CD Instance with Multiple Cluster Configurations:
      
      You only need one Argo CD installation in one cluster (often called the "primary" cluster).
      Argo CD uses Kubernetes cluster contexts to connect to and manage other clusters. You can configure each GKE cluster as a "cluster resource" within the Argo CD setup.
      This approach allows you to deploy and monitor applications across clusters from a single Argo CD UI, making management easier and more centralized.
      Configuring Additional Clusters in Argo CD:
      
      To add additional clusters, use argocd cluster add with the appropriate context for each GKE cluster.
      Each GKE cluster’s kubeconfig and authentication details should be accessible by Argo CD (either via kubeconfig context or a service account with sufficient permissions).
      Once configured, you can specify the target cluster for each application within Argo CD by setting the destination accordingly.
      Using Argo CD Agents (Optional):
      
      For more complex setups, you might want to install an Argo CD agent (like argocd-agent) on each target cluster. This can be beneficial if you need reduced latency or better fault tolerance by having agents in each cluster. However, this is not required if you don’t have performance constraints.
      Access Control and Security:
      
      Ensure that your Argo CD instance has appropriate RBAC permissions to manage resources in each target cluster. This often means configuring service accounts in each cluster to allow Argo CD to deploy resources securely.
      With this setup, you avoid the need to install multiple Argo CD instances, and you can centrally manage applications across multiple GKE clusters on various GCP projects.
      
# Areez provisioned a WFP project on GCP, which has organization-level restrictions set on it..

    Due to these restrictions, GCP has not exposed an external IP for the project, meaning that external traffic cannot directly access resources within it. Additionally, the policy does not allow the use of unauthenticated access in Cloud Run, preventing our frontend application (which was running on Cloud Run) from accepting traffic externally.
    
    Solution:
    
    After analyzing the setup, we observed that GCP allows the use of external IPs only through a load balancer. To work around this restriction, Areez deployed both the frontend and backend applications within the same GKE cluster. Previously, we deployed the frontend on Cloud Run and the backend on the GKE cluster. Now, with both services running in the same cluster, he exposed the frontend through a load balancer. This configuration allows us to direct external traffic to the load balancer, which then routes it to the deployment pods.
    
    Additionally, Areez successfully connected the frontend and backend to the database, ensuring that the database credentials were securely provided to the pods, and pods get the creds from gcp secret manager using workload identity. This setup resolved the access limitations and allowed smooth operation across our services.
    
    
    
## To set up a Jenkins pipeline on an AWS instance, follow these steps:

1. Set Up the AWS Environment
    
    Launch an EC2 instance:
    Go to AWS EC2 Dashboard > Launch Instance.
    Choose an instance type (t2.micro for testing).
    Select Amazon Linux or Ubuntu as the OS.
    Configure security groups to allow inbound SSH (port 22) and HTTP (port 8080).
    Access the instance:
    SSH into the instance using the key pair you specified during instance creation:

    ssh -i "path_to_your_key.pem" ec2-user@your_instance_public_ip

2. Install Jenkins on the EC2 Instance
    
    Update your instance:
  
    sudo yum update -y   # For Amazon Linux
    sudo apt update -y   # For Ubuntu
    Install Java (Jenkins requires Java):

    sudo yum install java-1.8.0-openjdk -y  # For Amazon Linux
    sudo apt install openjdk-11-jdk -y      # For Ubuntu
    Install Jenkins:

# Add Jenkins repository
    
    sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# Install Jenkins
    
    sudo yum install jenkins -y             # Amazon Linux
    sudo apt install jenkins -y             # Ubuntu
    
# Start Jenkins
  
    sudo systemctl start jenkins
    sudo systemctl enable jenkins
3. Access Jenkins and Configure It
    Open a web browser and navigate to http://your_instance_public_ip:8080.
    Unlock Jenkins by copying the initial admin password:
    
    sudo cat /var/lib/jenkins/secrets/initialAdminPassword
    Complete the Jenkins setup wizard, install suggested plugins, and create your admin user.

4. Set Up the Jenkins Pipeline
  
  Install AWS CLI (optional, if you’ll interact with AWS resources from Jenkins):

      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install
      Install Pipeline Plugins in Jenkins:
      Go to Jenkins Dashboard > Manage Jenkins > Manage Plugins.
      Search for and install "Pipeline" and "Git" plugins.
      Create a New Pipeline:
      Go to Dashboard > New Item, enter a name, select "Pipeline," and click OK.
      Configure the Pipeline:
      In the Pipeline section, you can use either:
      Pipeline script directly in Jenkins
      Jenkinsfile from a repository
      A basic pipeline script:
      groovy
      Copy code
      pipeline {
        agent any
        stages {
          stage('Clone Repository') {
            steps {
              git 'https://github.com/your-repo-url.git'
            }
          }
          stage('Build') {
            steps {
              echo 'Building...'
              // Add build commands, e.g., mvn clean install
            }
          }
          stage('Test') {
            steps {
              echo 'Running tests...'
              // Add test commands, e.g., mvn test
            }
          }
          stage('Deploy') {
            steps {
              echo 'Deploying...'
              // Deploy to AWS, e.g., upload files to S3
            }
          }
        }
      }
5. Secure Jenkins and Enable CI/CD
    
    Add SSH keys for secure Git access if needed.
    Set up credentials in Jenkins for secure storage of AWS keys and repository credentials.
    Configure notifications for build results using email or Slack integrations.






## how jenkins can deploy image on ec2 instance.. how can we make connection


    To deploy a Docker image to an EC2 instance using Jenkins, you'll typically need to do the following:

        Prepare the EC2 Instance for Deployment:
        
        Install Docker on the EC2 instance (if not already installed).
        Set up SSH access from Jenkins to the EC2 instance to allow remote deployments.
        Set Up Jenkins to Access the EC2 Instance:
        
        Configure SSH credentials in Jenkins so it can connect to the EC2 instance securely.
        Write a Jenkins Pipeline script to build, push, and deploy the Docker image on the EC2 instance.
        Here's a step-by-step guide on how to achieve this.
        
        Step 1: Prepare the EC2 Instance for Docker Deployments
        Install Docker on the EC2 instance:
        
        bash
        Copy code
        # SSH into the EC2 instance
        ssh -i "path_to_your_key.pem" ec2-user@your_instance_public_ip
        
        # Install Docker
        sudo yum update -y                   # For Amazon Linux
        sudo yum install docker -y
        sudo service docker start
        sudo usermod -a -G docker ec2-user   # Add ec2-user to the docker group to avoid needing sudo
        
        # Confirm Docker installation
        docker --version
        Configure the Security Group:
        
        Make sure the EC2 security group has open ports for SSH (port 22) and the application port (e.g., 80 or 8080) if you're deploying a web app.
        Set Up SSH Access from Jenkins:
        
        Generate an SSH key pair on the Jenkins server if you don’t already have one:
        bash
        Copy code
        ssh-keygen -t rsa -b 2048 -f ~/.ssh/jenkins_ec2_key
        Copy the public key (~/.ssh/jenkins_ec2_key.pub) to the ~/.ssh/authorized_keys file on the EC2 instance:
        bash
        Copy code
        ssh-copy-id -i ~/.ssh/jenkins_ec2_key.pub ec2-user@your_instance_public_ip
        Step 2: Configure Jenkins to Deploy the Docker Image
        Add the EC2 SSH Key in Jenkins:
        
        Go to Jenkins Dashboard > Manage Jenkins > Manage Credentials > Global.
        Add a new credential:
        Kind: SSH Username with Private Key
        Username: ec2-user
        Private Key: Enter directly (paste the contents of ~/.ssh/jenkins_ec2_key)
        Create the Jenkins Pipeline:
        
        In Jenkins, create a new Pipeline project.
        Configure the pipeline with a script to:
        Build the Docker image.
        Push the Docker image to a container registry (optional, e.g., Docker Hub, ECR).
        Deploy the Docker image to the EC2 instance.
        Step 3: Jenkins Pipeline Script
        Here’s an example of a Jenkins Pipeline script to deploy a Docker image to the EC2 instance:
        
        groovy
        Copy code
        pipeline {
            agent any
            environment {
                DOCKER_IMAGE = "your-docker-image:latest"
                SSH_CREDENTIALS_ID = "your-ssh-credentials-id"
                EC2_INSTANCE = "ec2-user@your_instance_public_ip"
            }
            stages {
                stage('Build Docker Image') {
                    steps {
                        script {
                            // Build Docker image
                            sh "docker build -t ${DOCKER_IMAGE} ."
                        }
                    }
                }
        
                stage('Push to Docker Registry') {
                    steps {
                        script {
                            // Login and push to Docker Hub (or another registry)
                            // Replace `docker login` with the appropriate registry command if needed
                            sh "docker login -u your-username -p your-password"
                            sh "docker push ${DOCKER_IMAGE}"
                        }
                    }
                }
        
                stage('Deploy to EC2') {
                    steps {
                        script {
                            // SSH into the EC2 instance and deploy the Docker image
                            sshagent([SSH_CREDENTIALS_ID]) {
                                sh """
                                ssh -o StrictHostKeyChecking=no ${EC2_INSTANCE} << EOF
                                    docker pull ${DOCKER_IMAGE}
                                    docker stop app || true
                                    docker rm app || true
                                    docker run -d --name app -p 80:80 ${DOCKER_IMAGE}
                                EOF
                                """
                            }
                        }
                    }
                }
            }
        }
        Step 4: Configure Docker Login Credentials (Optional)
        If you need to authenticate with Docker Hub or another registry, add the credentials in Jenkins and reference them in the script.
        
        Explanation of Each Stage
        Build Docker Image: Builds the Docker image from your project’s Dockerfile.
        Push to Docker Registry: Pushes the image to Docker Hub or another registry.
        Deploy to EC2:
        Connects to the EC2 instance over SSH using the configured credentials.
        Pulls the latest Docker image.
        Stops and removes any existing container with the name app.
        Runs the Docker image as a new container on the specified port.
        Testing the Pipeline
        Save and run the pipeline.
        Watch the pipeline stages to ensure each step completes without errors.
        After the pipeline runs successfully, visit http://your_instance_public_ip to access the application.

        summmay:

        ssh key jenkins ma generate ker k ec2 instance ko deni parti ha.. mean ssh connection create kerna hota ha jis sa jenkins ec2 ma deployment ker rha hoga..
        
## Terraform manage multiple environments

Managing multiple environments in Terraform can be streamlined using several strategies. Here’s a brief explanation of each approach:

1. Terraform Workspaces

  Terraform workspaces allow you to create multiple, isolated state files within a single configuration. This means you can manage different environments (e.g., dev, staging, production) without duplicating code. Each workspace has its own state file, so resources managed in one workspace do not interfere with those in others.

  Usage: After defining your configuration, you can switch environments using commands like terraform workspace new dev or terraform workspace select prod. Environment-specific variables can be referenced dynamically    within the same codebase, allowing for flexible, isolated deployments.

2. Environment-Specific Directories

    With environment-specific directories, you maintain separate folders for each environment (e.g., environments/dev, environments/prod). Each directory contains its own main.tf, variables.tf, and backend configurations as needed. This approach allows for complete customization of each environment, making it easier to control separate configurations or variables.
    
    Usage: When deploying, simply cd into the desired environment’s directory and run Terraform commands (terraform init, apply, etc.). This method can be easier for teams that need strong configuration isolation.

3. External Variable Management Tool

    You can use external tools, like Terragrunt or environment management systems (e.g., AWS Parameter Store or HashiCorp Vault), to manage environment-specific variables and configurations. This centralizes variable management, enhancing flexibility and reducing the need to duplicate code across environments.
    
    Usage: With Terragrunt, for example, you can specify shared configurations in a terragrunt.hcl file. This allows you to inherit and dynamically pull different variables per environment without modifying the main Terraform configuration.

Summary
    
    Each method has its strengths, from workspaces for simplicity to external tools for more complex, centralized management. The choice depends on your team’s needs for isolation, complexity, and flexibility in managing multiple Terraform environments.

## What is the difference b/w apache and nginx web server

    https://www.geeksforgeeks.org/difference-between-apache-and-nginx/

## command

  System Intelligence:
* htop - Like top, but actually useful
* df -h - Because disk space issues find you
* netstat -tulpn - Your network's story
* lsof - What's using that port?
* ps aux | grep - Finding that runaway process
* dmesg - Kernel's gossip channel

File Operations:
* find . -name - Your file search superhero
* tar -xvf - Unzip like a pro
* rsync - scp's smarter cousin
* sed -i - Stream editing wizard
* awk - Text manipulation magic
* grep -r - Find text like a detective

Container Life:
* docker stats - Container vital signs
* docker logs -f - Live container stories
* crictl pods - Kubernetes container whisperer
* kubectl get pods - K8s status check

Monitoring Magic:
* tail -f - Log watching party
* watch - Command on repeat
* vmstat - Memory tales
* iostat - Disk performance poetry

Network Ninja:
* curl -v - HTTP storyteller
* nc - Network swiss army knife
* dig - DNS detective
* ss - Socket statistics

Security Stuff:
* chmod - Permission painter
* chown - Ownership wizard
* openssl - Certificate craftsman
* ssh-keygen - Key creator

Process Control:
* systemctl - Service sorcery
* journalctl - Log time machine
* kill -9 - Process terminator
* nice - Priority painter    The nice command is useful for controlling process priority and managing system load by allowing less critical processes to yield more CPU time to more critical ones.

Performance Profiling:
* strace - System call spy       Purpose: strace is a diagnostic tool used to monitor the interactions between a process and the Linux kernel. It intercepts and records the system calls made by a process, along with their arguments and return values.  Usage: It’s useful for debugging and understanding how a program works, especially when you're trying to identify why it fails or behaves unexpectedly.

* tcpdump - Network packet poet    tcpdump is a command-line packet analyzer that captures network packets transmitted over a network interface. It can display the packets in real-time or save them for later analysis.
* sar - System activity reporter   sar is a tool that collects and reports various system activity information, including CPU usage, memory usage, I/O statistics, and network activity. It can provide historical data as well. 
* perf - Performance profiler      perf is a powerful performance analysis tool for Linux. It provides insights into CPU performance counters, tracepoints, and more. 

Text Wrangling:
* cut -d - Column collector
* sort | uniq -c - Pattern finder
* tr - Character changer
* wc -l - Line counter

File System:
* du -sh - Directory size detective
* fdisk -l - Disk detective
* mount - filesystem connector
* ln -s - Symlink sorcerer

Shell Shortcuts:
* history | grep - Command time machine
* !! - Last command replay
* ctrl+r - Reverse search magic
* alias - Command shortcut creator

Miscellaneous Mastery:
* tee - Output splitter
* xargs - Command multiplier
* at - Job scheduler
* screen/tmux - Terminal multiplexer


## what is the difference b/w ss , netstat , lsof

Purpose: ss is used to display detailed information about network sockets. It can show all established network connections, listening sockets, and even packet statistics.
    
    ss -tuln: List all listening TCP and UDP sockets.
    ss -s: Display summary statistics about the system's socket connections.
    ss -p: Show process information related to the connections.

Purpose: netstat provides information about network connections, routing tables, and network interfaces.

    netstat -tuln: Show all listening TCP and UDP ports.
    netstat -p: Show process details related to connections.
    netstat -i: Display statistics for each network interface.

Purpose: lsof lists all open files, including network sockets and ports, as everything in Linux is represented as a file (even network connections).

    lsof -i :80: Show processes using port 80.
    lsof -u [username]: List all open files by a specific user.
    lsof -i -n -P: List all network connections with numerical addresses and ports.

## what is the difference b/w sed and awk

  sed(stream editor) is great for straightforward for finding text and replacements.

      sed 's/foo/bar/g' input.txt > output.txt                    

  awk is ideal for structured data extraction and reporting, allowing more complex processing like handling fields in data.

      awk '{print $2}' input.txt

  here information is in input.txt file.


## How to pass Gcp serviceaccount key to pod.

    Our team has written a code, in that code developer has given a path like this /opt/service/secret.json, for making this sense that our code will get "secret.json file" from this path when the code will ran into the pod/container. for intracting gcp services like our case we do this for getting secret. 
    
    so we make serviceaccount in gcp, assign IAM permission on it, download serviceaccount key from gcp, now we need to give this key to our pod in this /opt/service/secret.json
    with secret.json name, so our code will get this key from this path inside.
    
    what we gona do for this.. we have create a configmap in kubernets with name serviceaccount, add data with filename secret.json and add key on configmap. now attach this configmap with pod using volumes and volume mount of mount path /opt/service/. In that way our pod will get secret.json file on path /opt/service.
    
    remember: Pod configuration file ma mount path dny sa us path k name ki directory pod ma auto create hojati ha...   

    and code ma jo file name dya ha key k lye, wo filename ko hi configmap ma bnana ha... or usma key rakni ha.. other wise pod error dye ga.. or verification k lye ap pod k under jaker us path ma key dekh sakhty hn.. or agr path crash ker rha ho tu ap, pod crash krny sa phily jakr b dekh sakhty hn, or debug container ko use ker k b crashpod k under jasakhty hn 


## How to get values.yaml file from running helm chart..
    
    helm list  -n airflow     ---> command to list the repo..   and get the repo name..
    helm get values airflow -n airflow > values.yaml
    
    command to get values.yaml using helm repo release-name

## How to install airflow in kuberetes using helm

    helm repo add apache-airflow https://airflow.apache.org
    helm upgrade --install airflow apache-airflow/airflow --namespace airflow --create-namespace --values ./gke_cluster/airflow/values.yaml

    for this you need to get values.yaml file from repo...

## how to verify that service can successfully select the deployment pod.

    for this you need to verify the below information
    
    - verify that service selector label and pod selector label should be same. by this service select the pod in deployment..
    - verify that service target port and deployment pod(contianer port) should be same.
    - verify that service gets the endpoint which is pod ip. Check the pod ip using command "k get pod/pod-name -o wide" and describe the service and match pod ip with service endpoint. if it same this means service get a correct end point...
     
    this these steps you get to know that service correctly attach with pod.

## Amazon S3 Access Policy Guide
     

     Amazon S3 (Simple Storage Service) is a highly scalable object storage service provided by AWS. To maintain secure and controlled access, AWS provides various policies and mechanisms to manage who can access S3 resources, under what conditions, and with what permissions. This document outlines commonly used access policies for Amazon S3.
      
      1. Granting Read-Only Permission to a Public Anonymous User
      Granting read-only access to anonymous (public) users is useful for assets that need to be accessible by anyone, such as images or documents for a website.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example-bucket/*"
          }
        ]
      }
      Use Case: Public content like website assets.
      Note: Use this policy carefully, as it grants public read-only access to all objects in the bucket.
      2. Requiring Encryption for Objects
      This policy enforces that all objects stored in the bucket are encrypted, enhancing security and compliance.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:PutObject",
            "Resource": "arn:aws:s3:::example-bucket/*",
            "Condition": {
              "StringNotEquals": {
                "s3:x-amz-server-side-encryption": "AES256"
              }
            }
          }
        ]
      }
      Use Case: Organizations requiring encryption for regulatory compliance.
      Supported Encryption: AES256, aws:kms.
      3. Managing Buckets Using Canned ACLs
      Canned ACLs (Access Control Lists) are predefined access control settings for buckets and objects. They provide a simple way to manage permissions without complex policies.
      
      Common ACLs
      private: Owner has full control.
      
      public-read: Objects are readable by everyone.
      
      public-read-write: Objects are readable and writable by everyone.
      
      Use Case: Basic access management for public or internal resources.
      
      4. Managing Object Access with Object Tagging
      By applying tags to objects, you can control access based on tags. This enables policies that restrict or allow access to objects with specific tags, helping in categorizing and securing data.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example-bucket/*",
            "Condition": {
              "StringEquals": {
                "s3:ExistingObjectTag/department": "finance"
              }
            }
          }
        ]
      }
      Use Case: Restrict access to data based on department or data classification tags.
      5. Managing Object Access by Using Global Condition Keys
      Global condition keys allow you to set access conditions based on criteria such as IP addresses, request time, or other factors.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Deny",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::example-bucket/*",
            "Condition": {
              "NotIpAddress": {
                "aws:SourceIp": "192.0.2.0/24"
              }
            }
          }
        ]
      }
      Use Case: Restrict access to specific IP ranges, such as company networks.
      6. Managing Access Based on HTTP or HTTPS Requests
      To ensure data is securely transmitted, you can enforce HTTPS access.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::example-bucket/*",
            "Condition": {
              "Bool": {
                "aws:SecureTransport": "false"
              }
            }
          }
        ]
      }
      Use Case: Enforce HTTPS to secure data transmission.
      7. Managing User Access to Specific Folders
      This policy allows you to grant access to specific folders within a bucket. Useful for multi-user environments.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example-bucket/folder1/*"
          }
        ]
      }
      Use Case: Departmental access control.
      8. Managing Access for Access Logs
      To maintain access control over logs, grant designated users permission to read or manage access logs.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": { "AWS": "arn:aws:iam::account-id:user/LogManager" },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example-bucket/logs/*"
          }
        ]
      }
      Use Case: Grant access to logs for monitoring or auditing.
      9. Managing Access to an Amazon CloudFront Origin Access Identity (OAI)
      Use an OAI to restrict direct access to S3 content and allow access only through CloudFront.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": {
              "AWS": "arn:aws:iam::cloudfront-user-id"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example-bucket/*"
          }
        ]
      }
      Use Case: Secure content by requiring access only through CloudFront.
      10. Managing Access for Amazon S3 Storage Lens
      S3 Storage Lens provides visibility into S3 usage. You can control access to Storage Lens reports and insights with policies.
      
      Use Case: Grant Storage Lens access for usage monitoring.
      11. Managing Permissions for S3 Inventory, Analytics, and Reports
      This policy controls access to S3 Inventory and analytics reports.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": { "AWS": "arn:aws:iam::account-id:user/InventoryUser" },
            "Action": [
              "s3:GetObject"
            ],
            "Resource": "arn:aws:s3:::example-bucket/inventory-reports/*"
          }
        ]
      }
      Use Case: Grant permissions to access S3 data analytics.
      12. Requiring Multi-Factor Authentication (MFA)
      Require MFA for sensitive actions, such as object deletion.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Deny",
            "Action": "s3:DeleteObject",
            "Resource": "arn:aws:s3:::example-bucket/*",
            "Condition": {
              "Bool": {
                "aws:MultiFactorAuthPresent": "false"
              }
            }
          }
        ]
      }
      Use Case: Prevent deletions unless MFA is used.
      13. Preventing Users from Deleting Objects
      To enforce data retention, deny deletion of objects in the bucket.
      
      Example Policy
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Deny",
            "Action": "s3:DeleteObject",
            "Resource": "arn:aws:s3:::example-bucket/*"
          }
        ]
      }
      Use Case: Enforce data retention requirements.
      These policies provide a range of access management options to enhance security, control, and compliance for S3 resources.

## If someone ask you to deploy db on cluster, what should be the db kind. deployment or statefulset.

    
    When deploying a database on a Kubernetes cluster, **a StatefulSet is the preferred choice rather than a Deployment**. Here’s why:
    
    Why Use StatefulSet for Databases?
    Stable Network Identity:
    
    StatefulSets provide each replica of a pod with a unique, stable network identity (like db-0, db-1, etc.). This is critical for databases, where each instance often needs to be accessed individually, especially in clustered setups like MySQL or PostgreSQL replicas.
    Persistent Storage:
    
    StatefulSets support persistent storage with PersistentVolumeClaims (PVCs) that are bound to each pod. This ensures that data persists even if the pod is rescheduled or restarted. Deployment, on the other hand, does not guarantee persistent storage since each pod instance is created as a standalone entity.
    Ordered, Graceful Scaling and Updates:
    
    StatefulSets provide ordered scaling and deployment, ensuring that pods are terminated and brought up in sequence. This feature is essential for databases because it ensures that one instance stabilizes before the next instance starts, maintaining data integrity.
    Handling Failovers and Data Consistency:
    
    With databases, handling failover and data consistency is critical. A StatefulSet’s ordered startup and shutdown process supports consistency and data recovery in case of failures.
    When to Consider Deployment Instead
    In rare cases, if the database is fully stateless (such as an in-memory database or a temporary caching layer like Redis without persistence), a Deployment might be sufficient. However, for traditional databases requiring data persistence, consistency, and stable identifiers, a StatefulSet is the best choice.
    
    Example of StatefulSet for Database
    Here’s a simplified example of a StatefulSet YAML for a MySQL database:
    
    yaml
    Copy code
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: mysql
    spec:
      serviceName: mysql
      replicas: 3
      selector:
        matchLabels:
          app: mysql
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - name: mysql
            image: mysql:5.7
            ports:
            - containerPort: 3306
            volumeMounts:
            - name: mysql-data
              mountPath: /var/lib/mysql
      volumeClaimTemplates:
      - metadata:
          name: mysql-data
        spec:
          accessModes: ["ReadWriteOnce"]
          storageClassName: "standard"
          resources:
            requests:
              storage: 10Gi

    This configuration ensures that each MySQL instance in the cluster has its own unique identity and persistent storage.
 
    For my understanding, I think deployment is best for the application, In which request will loadbalance on each pod, but in case database deployment in cluster, you should use statefulset, because you need to make this sure that only write request will send on master pod, and read request will send on master and slave both. statefullset handle this easily, because it use the handless service, which provides the unique identity to the pod, like on master db-0 and on slave db-1. 

    And statefulset is maintain his state, mean agr pod restart b hoga ha tu iska network, podname stable rhy ga... or data b pod ma persist rhta ha.. jb k deployment ma data persist ni rhta, on each pod restart... it also make sure data consistency and data recovery on each failure.


## Linux Commands

  1. ls: List directory contents. 
  2. cd: Change directory.
  3. pwd: Print working directory. 
  4. mkdir: Create a directory.
  5. touch: Create a file.
  6. cp: Copy files and directories.
  7. mv: Move or rename files and directories. 
  8. rm: Remove files and directories.
  9. find: Search for files and directories. 
  10. grep: Search for patterns in files.
  11. cat: Concatenate and display files.
  12. less: View file contents page by page. 
  13.head: Display the first lines of a file.
  14. tail: Display the last lines of a file. 
  15. vi/vim: Text editor. 
  16. nano: Text editor.
  17. tar: Archive and compress files. 
  18. gzip: Compress files.
  19. gunzip: Decompress files.
  20. wget: Download files from the web.
  21. curl: Transfer data to or from a server. 
  22. ssh: Secure shell remote login.
  23. scp: Securely copy files between hosts.
  24. chmod: Change file permissions. 
  25. chown: Change file ownership.
  26. chgrp: Change group ownership. 
  27. ps: Display running processes.
  28. top: Monitor system resources and processes.
  29. kill: Terminate processes.
  30. df: Display disk space usage.
  31. du: Estimate file and directory space usage. 
  32. free: Display memory usage.
  33. uname: Print system information.
  34. ifconfig: Configure network interfaces.
  35. ping: Test network connectivity. 
  36. netstat: Network statistics.
  37. iptables: Firewall administration.
  38. systemctl: Manage system services.
  39. journalctl: Query the system journal. 
  40. crontab: Schedule cron jobs.
  41. useradd: Create a user account.
  42. passwd: Change user password.
  43. su: Switch user.
  44. sudo: Execute a command as another user. 
  45. usermod: Modify user account. 
  46. groupadd: Create a group.
  47. groupmod: Modify a group.
  48. id: Print user and group information. 
  49. ssh-keygen: Generate SSH key pairs.
  50. rsync: Synchronize files and directories. 
  51. diff: Compare files line by line. 
  52. patch: Apply a patch to files.
  53. tar: Extract files from an archive. 
  54. curl: Perform HTTP requests.
  55. nc: Netcat - networking utility. 
  56. wget: Download files from the web.
  57. whois: Lookup domain registration details. 
  58. dig: DNS lookup utility.
  59. sed: Stream editor for text manipulation.
  60. awk: Pattern scanning and processing language. 
  61. sort: Sort lines in a text file.
  62. cut: Extract sections from lines of files.
  63. wc: Word, line, character, and byte count.
  64. tee: Redirect output to multiple files or commands.
  65. history: Command history.
  66. source: Execute commands from a file in the current shell. 
  67. alias: Create command aliases.
  68. ln: Create links between files.
  69. uname: Print system information.
  70. lsof: List open files and processes.


## Linux concepts as a beginner. 
    
    1. What is the boot process in Linux?
    The Linux boot process has several stages:
    
    BIOS/UEFI: Power on, BIOS/UEFI checks hardware and passes control to the bootloader.
    Bootloader: (GRUB or LILO) Loads the Linux kernel.
    Kernel: The kernel initializes and loads essential services.
    Init/Systemd: Manages user space and starts services.
    Login: Shows the login prompt or graphical user interface (GUI).
    2. How can you create a zero-size file in Linux?
    Use the touch command: touch filename.txt
    3. What are soft links and hard links in Linux?
    Soft Link (Symbolic Link): Points to the original file. Like a shortcut.
    Command: ln -s original_file link_name
    Hard Link: Points to the data on disk. Deleting the original file doesn't affect it.
    Command: ln original_file link_name
    Difference: Hard links can't be used across different file systems or directories, while soft links can.
    4. What is the first line typically written in a shell script?
    The first line is #!/bin/bash, called the "shebang."
    It tells the system which interpreter (like bash) to use.
    Without it, you may need to run scripts as bash script.sh instead of ./script.sh.
    5. How can you run a shell script in the background in Linux?
    Add an & at the end of the command: ./script.sh &
    6. What is a crontab in Linux?
    Crontab schedules recurring tasks.
    Command to edit: crontab -e
    Syntax example: * * * * * /path/to/script.sh (Runs every minute)
    7. How do you allow ports in Linux?
    For Ubuntu/Debian: sudo ufw allow port_number
    For CentOS/RHEL: sudo firewall-cmd --add-port=port_number/tcp --permanent
    8. How do you troubleshoot a remote server that is experiencing issues?
    Ping the server: ping server_ip
    SSH into the server: ssh user@server_ip
    Check logs: tail -f /var/log/syslog
    9. What are the ping, telnet, curl, and wget commands?
    ping: Checks network connectivity.
    telnet: Tests connectivity to specific ports.
    curl: Requests data from or sends data to URLs.
    wget: Downloads files from the internet.
    10. How can you check the status of services in a Linux machine?
    systemctl status service_name (for newer systems)
    11. How do you kill a process in Linux?
    Find the process ID: ps aux | grep process_name
    Kill the process: kill process_id
    12. What are the nice and renice commands in Linux?
    nice: Sets the priority of a new process.
    renice: Changes the priority of a running process.
    13. What is an inode in Linux?
    An inode stores information about a file or directory, such as its size, owner, and permissions.
    14. How do you check CPU utilization in Linux?
    Use the top or htop commands.
    15. What are the differences between the top and htop commands?
    top: Command-line process viewer.
    htop: Enhanced version with a visual interface. Easier to read.
    16. What is a mount in Linux, and how do you create one?
    Mounting connects a storage device or partition to a directory.
    Example: sudo mount /dev/sdb1 /mnt
    17. How do you troubleshoot live logs in Linux?
    Use tail -f /var/log/syslog to view live logs.
    18. What is the sed command in Linux?
    sed is a stream editor for text manipulation.
    Example: sed 's/old/new/g' file.txt
    19. What is the awk command in Linux?
    awk processes and analyzes text, often used with columns.
    Example: awk '{print $1}' file.txt (Prints the first column)
    20. What are the grep and egrep commands in Linux?
    grep: Searches for text patterns.
    egrep: Extended version that supports more patterns.
    21. How can you list only directories in a Linux environment?
    Use ls -d */
    22. How do you check the processes running in Linux?
    Use the ps aux command or top command.
    23. How do you get a Java thread dump in Linux?
    Use the jstack process_id command.
    24. How can you check the running ports on a Linux machine?
    Use netstat -tuln or ss -tuln.
    25. How do you declare a variable in a shell script?
    Syntax: variable_name=value
    26. What do $?, $#, and $ represent in shell scripting?*
    $?: Last command's exit status.
    $#: Number of arguments passed.
    $*: All arguments passed to the script.
    27. How do you read a command line input in a shell script?
    Use read variable_name
    28. What is umask in Linux?
    umask sets default file permissions.
    Command: umask 022
    29. How do you change file permissions in Linux?
    Use chmod permissions file_name (e.g., chmod 755 script.sh)
    30. How can you connect to remote servers without a password?
    Use SSH keys:
    Generate a key: ssh-keygen
    Copy the key: ssh-copy-id user@server_ip
    31. How do you open a file in read-only mode in the vi editor?
    Command: vi -R filename
    32. What is the purpose of the export command in Linux?
    export makes a variable available to child processes.
    Example: export PATH=$PATH:/new/path
    33. How do you send error logs and stdout logs to different files in Linux?
    Command: command > output.log 2> error.log
    34. What is the nohup command in Linux?
    nohup keeps a command running even after logging out.
    Usage: nohup command &
    35. What does the netstat command do in Linux?
    netstat shows network connections, routing tables, and listening ports.
    36. How do you run a script at boot level in Linux?
    Add it to /etc/rc.local or create a systemd service


## AWS CloudTrail

    AWS CloudTrail is a service that provides comprehensive monitoring and logging for actions taken on your AWS account. 
    
    Audit Logging: CloudTrail records actions taken by users, roles, and services, capturing API calls and other events that could impact the security of your AWS account. It’s crucial for auditing purposes, as it logs details about who made a request, when it was made, and what changes were made.
    Compliance Requirements: Many compliance standards (such as HIPAA, GDPR, and SOC) require detailed logging of all actions within the cloud environment. CloudTrail helps meet these requirements by maintaining a comprehensive audit trail.
    Detecting Unauthorized Access: By monitoring CloudTrail logs, you can detect unauthorized or suspicious activities, such as access from unfamiliar IP addresses, unusual usage patterns, or attempts to access restricted resources.

## To remove a package but keep its configuration files on a Linux system, you can use the following commands depending on the package manager you are using:

    On Debian/Ubuntu (APT)
    Use the **remove** command instead of **purge**. The remove command will uninstall the package without deleting its configuration files.


    sudo apt remove <package_name>

## site for learning terraform content

    https://developer.hashicorp.com/terraform/tutorials/aws-get-started/infrastructure-as-code

## terraform reusibility

    Here’s a basic example of using modules in Terraform to create reusable infrastructure components, like a VPC, subnet, route table, and security groups. This setup will show how to organize your code so you can reuse the modules across different environments or configurations.
    
    Step 1: Set up the directory structure
    Create a directory structure like this:
    
    plaintext
    Copy code
    project-root/
    ├── main.tf
    ├── modules/
    │   ├── vpc/
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   │   └── variables.tf
    │   ├── subnet/
    │   │   ├── main.tf
    │   │   ├── outputs.tf
    │   │   └── variables.tf
    │   └── security_group/
    │       ├── main.tf
    │       ├── outputs.tf
    │       └── variables.tf
    └── variables.tf
    Step 2: Define the VPC Module
    In modules/vpc/main.tf, define the VPC resource:
    
    hcl
    Copy code
    // modules/vpc/main.tf
    resource "aws_vpc" "this" {
      cidr_block = var.cidr_block
      tags = {
        Name = var.name
      }
    }
    In modules/vpc/variables.tf, define the input variables:
    
    hcl
    Copy code
    // modules/vpc/variables.tf
    variable "cidr_block" {
      description = "The CIDR block for the VPC"
      type        = string
    }
    
    variable "name" {
      description = "The name tag for the VPC"
      type        = string
      default     = "example-vpc"
    }
    In modules/vpc/outputs.tf, define the output:
    
    hcl
    Copy code
    // modules/vpc/outputs.tf
    output "vpc_id" {
      value = aws_vpc.this.id
    }
    Step 3: Define the Subnet Module
    In modules/subnet/main.tf, create the subnet resource:
    
    hcl
    Copy code
    // modules/subnet/main.tf
    resource "aws_subnet" "this" {
      vpc_id            = var.vpc_id
      cidr_block        = var.cidr_block
      availability_zone = var.availability_zone
      tags = {
        Name = var.name
      }
    }
    Define input variables in modules/subnet/variables.tf:
    
    hcl
    Copy code
    // modules/subnet/variables.tf
    variable "vpc_id" {
      description = "The ID of the VPC"
      type        = string
    }
    
    variable "cidr_block" {
      description = "The CIDR block for the subnet"
      type        = string
    }
    
    variable "availability_zone" {
      description = "The availability zone for the subnet"
      type        = string
    }
    
    variable "name" {
      description = "The name tag for the subnet"
      type        = string
      default     = "example-subnet"
    }
    Define outputs in modules/subnet/outputs.tf:
    
    hcl
    Copy code
    // modules/subnet/outputs.tf
    output "subnet_id" {
      value = aws_subnet.this.id
    }
    Step 4: Define the Security Group Module
    In modules/security_group/main.tf, create a security group resource:
    
    hcl
    Copy code
    // modules/security_group/main.tf
    resource "aws_security_group" "this" {
      name        = var.name
      description = var.description
      vpc_id      = var.vpc_id
    
      ingress {
        from_port   = var.ingress_from_port
        to_port     = var.ingress_to_port
        protocol    = var.ingress_protocol
        cidr_blocks = var.ingress_cidr_blocks
      }
    
      egress {
        from_port   = 0
        to_port     = 0
        protocol    = "-1"
        cidr_blocks = ["0.0.0.0/0"]
      }
    }
    Define input variables in modules/security_group/variables.tf:
    
    hcl
    Copy code
    // modules/security_group/variables.tf
    variable "name" {
      description = "The name of the security group"
      type        = string
    }
    
    variable "description" {
      description = "The description of the security group"
      type        = string
    }
    
    variable "vpc_id" {
      description = "The ID of the VPC"
      type        = string
    }
    
    variable "ingress_from_port" {
      description = "Ingress from port"
      type        = number
    }
    
    variable "ingress_to_port" {
      description = "Ingress to port"
      type        = number
    }
    
    variable "ingress_protocol" {
      description = "Ingress protocol"
      type        = string
    }
    
    variable "ingress_cidr_blocks" {
      description = "Ingress CIDR blocks"
      type        = list(string)
    }
    Define outputs in modules/security_group/outputs.tf:
    
    hcl
    Copy code
    // modules/security_group/outputs.tf
    output "security_group_id" {
      value = aws_security_group.this.id
    }

    Step 5: Use the Modules in main.tf

    Now that the modules are defined, use them in the main main.tf to create a reusable setup:
    
    hcl
    Copy code
    // main.tf
    
    provider "aws" {
      region = "us-west-2"
    }
    
    module "vpc" {
      source     = "./modules/vpc"
      cidr_block = "10.0.0.0/16"
      name       = "my-vpc"
    }
    
    module "subnet" {
      source            = "./modules/subnet"
      vpc_id            = module.vpc.vpc_id
      cidr_block        = "10.0.1.0/24"
      availability_zone = "us-west-2a"
      name              = "my-subnet"
    }
    
    module "security_group" {
      source              = "./modules/security_group"
      vpc_id              = module.vpc.vpc_id
      name                = "my-security-group"
      description         = "Security group for my application"
      ingress_from_port   = 22
      ingress_to_port     = 22
      ingress_protocol    = "tcp"
      ingress_cidr_blocks = ["0.0.0.0/0"]
    }

    Step 6: Run Terraform Commands
    
    Initialize Terraform:
    
    terraform init
    Apply the configuration:


    terraform apply
    This configuration demonstrates reusability with modules in Terraform. You can now reuse these modules across different projects or environments by adjusting variables as needed.

    **module name ki directory ma apna apny reusible resource ko rakh dena ha, or phir sab other files like (main.tf) ma apna in module ko call krna ha.** is tarha sa ap terraform ma reusibilty ko apply ker sakhty hn.
    

## AWS CROSS ACCOUNT ACCESS

      To allow a user in one AWS account to access resources (like an EC2 instance or S3 bucket) in another AWS account, you can set up cross-account access using AWS Identity and Access Management (IAM) roles. Here’s a step-by-step guide:
      
      1. Define the Two Accounts
      Account A: The account that owns the resources (like the EC2 instance or S3 bucket).
      Account B: The account with the user or role that needs access to resources in Account A.
      2. Set Up an IAM Role in Account A (Resource Account)
      Log in to Account A (the resource account).
      Go to the IAM Console.
      Choose Roles in the left sidebar, then click Create role.
      Under Select trusted entity, select Another AWS account.
      Enter the Account ID of Account B.
      Enable Require external ID (optional but recommended for security). Enter a unique external ID that the user in Account B will use when accessing resources.
      Click Next to attach permissions.
      3. Attach Permissions to the IAM Role
      Attach a policy to the IAM role in Account A that specifies which resources in Account A the user in Account B can access. Here are two examples:
      
      Example A: Grant Access to an EC2 Instance
      Choose Create policy or attach an existing policy.
      
      Use a policy similar to this to allow access to an EC2 instance:
      
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "ec2:DescribeInstances",
              "ec2:StartInstances",
              "ec2:StopInstances"
            ],
            "Resource": "arn:aws:ec2:region:AccountA_ID:instance/instance-id"
          }
        ]
      }
      Example B: Grant Access to an S3 Bucket
      Use a policy similar to this to allow access to an S3 bucket:
      
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "s3:GetObject",
              "s3:ListBucket"
            ],
            "Resource": [
              "arn:aws:s3:::bucket-name",
              "arn:aws:s3:::bucket-name/*"
            ]
          }
        ]
      }
      Click Next and Review. Give the role a name (like AccountBAccessRole) and Create role.
      
      4. Note the ARN of the Role
      After creating the role, note the Role ARN (e.g., arn:aws:iam::AccountA_ID:role/AccountBAccessRole) because you'll need it for configuring Account B.
      
      5. Configure Access in Account B (User Account)
      Log in to Account B.
      
      Go to the IAM Console and navigate to the Users or Roles section (depending on how access is granted in Account B).
      
      Edit the User or Role permissions in Account B and attach a policy that allows the user to assume the role created in Account A.
      
      json
      Copy code
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": "sts:AssumeRole",
            "Resource": "arn:aws:iam::AccountA_ID:role/AccountBAccessRole"
          }
        ]
      }
      6. Assume the Role in Account B
      To access resources in Account A, the user in Account B must assume the role via the AWS CLI or SDK.
      
      Using AWS CLI:
      
      bash
      Copy code
      aws sts assume-role --role-arn "arn:aws:iam::AccountA_ID:role/AccountBAccessRole" --role-session-name "AccountBUserSession"
      This command will return temporary credentials for accessing resources in Account A.
      
      7. Verify Access
      Using the temporary credentials returned by the assume-role command, the user in Account B can now access the resources in Account A according to the permissions granted in the role's policy.
      
      This setup allows secure cross-account access between two AWS accounts without sharing credentials directly.

## HOW CAN JENKINS GET ACCESS OF GCP CLOUD RESOURCE USING SERVICE ACCOUNT

      Yes, Jenkins can access Google Cloud Platform (GCP) resources using a Google Cloud service account. Here’s how you can set it up:
      
      1. Create a Service Account in GCP
      Go to the GCP Console.
      Navigate to IAM & Admin > Service Accounts.
      Click Create Service Account.
      Give it a name and description that helps identify its purpose.
      Under Grant this service account access to the project, assign the necessary roles (e.g., Viewer, Storage Admin, Compute Admin, etc.), depending on what resources Jenkins needs to access.
      Click Continue, and Done to finish creating the service account.
      2. Generate a Key for the Service Account
      In the Service Accounts section, find the service account you just created.
      Click on the Actions (three dots) next to the service account and select Manage Keys.
      Click Add Key > Create New Key.
      Select JSON as the key type and click Create. This will download a JSON key file to your computer. Keep this file secure, as it allows access to your GCP resources.
      3. Configure Jenkins to Use the Service Account Key
      Go to your Jenkins instance.
      In Jenkins, navigate to Manage Jenkins > Manage Credentials.
      Add the JSON key file as a credential:
      Select the correct domain (usually Global).
      Click Add Credentials.
      Select Secret file as the Kind.
      Choose the JSON key file you downloaded, and give it a meaningful ID (like gcp-service-account).
      4. Use the Service Account in Your Jenkins Pipeline
      In your Jenkins pipeline script, you can now reference this credential to authenticate with GCP. Here’s how to set up authentication in your pipeline using the GOOGLE_APPLICATION_CREDENTIALS environment variable.
      
      groovy
      Copy code
      pipeline {
          agent any
      
          environment {
              GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-service-account')  // Use the ID you set earlier
          }
      
          stages {
              stage('Access GCP') {
                  steps {
                      script {
                          // Example: Run gcloud commands
                          sh 'gcloud auth activate-service-account --key-file="$GOOGLE_APPLICATION_CREDENTIALS"'
                          
                          // Now you can run gcloud or other GCP SDK commands
                          sh 'gcloud compute instances list'
                      }
                  }
              }
          }
      }
      Alternative: Use a Plugin for GCP Authentication
      Jenkins has plugins like the Google OAuth Credentials plugin that can handle GCP authentication more seamlessly. The plugin also allows setting up authentication with the service account directly from Jenkins without scripting the setup.
      
      By following these steps, Jenkins should be able to securely access GCP resources using a service account.  

## Cannot access global variable in  jenkins pipeline from jenkins global variable setting under manage jenkins

    Remember: jenkins ma jb ap **manage jenkins ma > system** option ma jakr global variable ko set krty hn, tky y global variable sab pipeline ma access ho sakhy or ap inhy pipeline ma khi b use kr sakhty **tu ap na variable k names ma -(hifen) ki jaga _(underscore) use krna ha other wise jenkins pipeline fail hojy gi because wo global variable sa variable ko access ni kr rhi hogi..** 

## jenkins example pipelines

      gcloud config set account ${disearchmt-dev} && gcloud config set project disearchmt-dev  ---> this command will use service account for cloud authenticate.


    RELEASE_VERSION=1.0.38-Hotfix
    gcloud config set account ${disearch} && gcloud config set project disearch
    docker pull gcr.io/disearch/vertexai-citation:latest
    
    docker tag gcr.io/disearch/vertexai-citation:latest gcr.io/disearch-vertexai/vertexai-citation:$RELEASE_VERSION
    docker tag gcr.io/disearch/vertexai-citation:latest gcr.io/pcpeprod/vertexai-citation:$RELEASE_VERSION
    docker tag gcr.io/disearch/vertexai-citation:latest gcr.io/diseracharetecprod/vertexai-citation:$RELEASE_VERSION
    docker tag gcr.io/disearch/vertexai-citation:latest gcr.io/lumos-disearch-st/vertexai-citation:$RELEASE_VERSION
    docker tag gcr.io/disearch/vertexai-citation:latest gcr.io/cdc-oman-st/vertexai-citation:$RELEASE_VERSION
    
    gcloud config set account ${disearch_vertexai} && gcloud config set project disearch-vertexai
    docker push gcr.io/disearch-vertexai/vertexai-citation:$RELEASE_VERSION
    gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project disearch-vertexai
    kubectl set image deployment/vertexai-citation-deployment vertexai-citation=gcr.io/disearch-vertexai/vertexai-citation:$RELEASE_VERSION
    
    gcloud config set account ${pcpeprod} && gcloud config set project pcpeprod
    docker push gcr.io/pcpeprod/vertexai-citation:$RELEASE_VERSION
    gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project pcpeprod
    kubectl set image deployment/vertexai-citation-deployment vertexai-citation=gcr.io/pcpeprod/vertexai-citation:$RELEASE_VERSION
    
    gcloud config set account ${diseracharetecprod} && gcloud config set project diseracharetecprod
    docker push gcr.io/diseracharetecprod/vertexai-citation:$RELEASE_VERSION
    gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project diseracharetecprod
    kubectl set image deployment/vertexai-citation-deployment vertexai-citation=gcr.io/diseracharetecprod/vertexai-citation:$RELEASE_VERSION
    
    gcloud config set account ${lumos_disearch_st} && gcloud config set project lumos-disearch-st
    docker push gcr.io/lumos-disearch-st/vertexai-citation:$RELEASE_VERSION
    gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project lumos-disearch-st
    kubectl set image deployment/vertexai-citation-deployment vertexai-citation=gcr.io/lumos-disearch-st/vertexai-citation:$RELEASE_VERSION
    
    gcloud config set account ${cdc_oman_st} && gcloud config set project cdc-oman-st
    docker push gcr.io/cdc-oman-st/vertexai-citation:$RELEASE_VERSION
    gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project cdc-oman-st
    kubectl set image deployment/vertexai-citation-deployment vertexai-citation=gcr.io/cdc-oman-st/vertexai-citation:$RELEASE_VERSION
    

    gcloud config set account ${disearch} && gcloud config set project disearch
    docker build -f Dockerfile -t gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT .
    docker tag gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT gcr.io/disearch/multi-model-doc-chat:latest
    docker push gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT
    docker push gcr.io/disearch/multi-model-doc-chat:latest
    gcloud container clusters get-credentials search-app-dev --zone us-central1-c --project disearch
    kubectl set image deployment/multi-model-doc-chat-deployment multi-model-doc-chat=gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT
    gcloud run deploy multi-model-doc-chat \
    --image=gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT \
    --set-env-vars= \
    --set-secrets=DB_HOST=DB_HOST:latest,DB_PASSWORD=DB_PASSWORD:latest,DB_USER=DB_USER:latest,DB_SCHEMA=DB_SCHEMA:latest,MULTI_MODEL_GCP_BUCKET=MULTI_MODEL_GCP_BUCKET:latest,vertexai-referer=vertexai-referer:latest \
    --region=us-east1 \
    --project=disearch
    gcloud run services update-traffic multi-model-doc-chat --to-latest --region us-east1
    

    gcloud config set account ${disearchmt_dev} && gcloud config set project disearchmt-dev
    docker tag gcr.io/disearch/multi-model-doc-chat:dev-$GIT_COMMIT gcr.io/disearchmt-dev/multi-model-doc-chat:dev-$GIT_COMMIT
    docker push gcr.io/disearchmt-dev/multi-model-doc-chat:dev-$GIT_COMMIT
    gcloud container clusters get-credentials disearch-cluster --zone us-central1-c --project disearchmt-dev
    kubectl set image deployment/multi-model-doc-chat-deployment multi-model-doc-chat=gcr.io/disearchmt-dev/multi-model-doc-chat:dev-$GIT_COMMIT
    gcloud run deploy multi-model-doc-chat \
    --image=gcr.io/disearchmt-dev/multi-model-doc-chat:dev-$GIT_COMMIT \
    --set-env-vars= \
    --set-secrets=DB_HOST=DB_HOST:latest,DB_USER=DB_USER:latest,DB_PASSWORD=DB_PASSWORD:latest,DB_SCHEMA=DB_SCHEMA:latest,MULTI_MODEL_GCP_BUCKET=MULTI_MODEL_GCP_BUCKET:latest,vertexai-referer=vertexai-referer:latest \
    --region=us-central1 \
    --project=disearchmt-dev 
    gcloud run services update-traffic multi-model-doc-chat --to-latest --region=us-central1
    RELEASE_VERSION=1.0.40

    gcloud config set account ${disearch} && gcloud config set project disearch
    docker build -f Dockerfile -t gcr.io/disearch/multi-model-doc-chat:$RELEASE_VERSION .
    
    
    docker tag gcr.io/disearch/multi-model-doc-chat:$RELEASE_VERSION gcr.io/pcpeprod/multi-model-doc-chat:$RELEASE_VERSION
    docker tag gcr.io/disearch/multi-model-doc-chat:$RELEASE_VERSION gcr.io/cdc-oman-st/multi-model-doc-chat:$RELEASE_VERSION
    docker tag gcr.io/disearch/multi-model-doc-chat:$RELEASE_VERSION gcr.io/disearch-vertexai/multi-model-doc-chat:$RELEASE_VERSION
    docker tag gcr.io/disearch/multi-model-doc-chat:$RELEASE_VERSION gcr.io/diseracharetecprod/multi-model-doc-chat:$RELEASE_VERSION
    docker tag gcr.io/disearch/multi-model-doc-chat:$RELEASE_VERSION gcr.io/lumos-disearch-st/multi-model-doc-chat:$RELEASE_VERSION
    docker tag gcr.io/disearch/multi-model-doc-chat:$RELEASE_VERSION gcr.io/world-learning-400909/multi-model-doc-chat:$RELEASE_VERSION
    
    
    gcloud config set account ${pcpeprod} && gcloud config set project pcpeprod
    docker push gcr.io/pcpeprod/multi-model-doc-chat:$RELEASE_VERSION
    gcloud run deploy multi-model-doc-chat \
    --image=gcr.io/pcpeprod/multi-model-doc-chat:$RELEASE_VERSION \
    --allow-unauthenticated \
    --port=8000 \
    --service-account=913855764051-compute@developer.gserviceaccount.com \
    --min-instances=1 \
    --set-env-vars= \
    --set-cloudsql-instances=pcpeprod:us-central1:disearch-db \
    --vpc-connector=projects/pcpeprod/locations/us-central1/connectors/disearch-vpc-connector \
    --vpc-egress=private-ranges-only \
    --set-secrets=DB_HOST=DB_HOST:latest,DB_USER=DB_USER:latest,DB_PASSWORD=DB_PASSWORD:latest,DB_SCHEMA=DB_SCHEMA:latest,vertexai-referer=vertexai-referer:latest,MULTI_MODEL_GCP_BUCKET=MULTI_MODEL_GCP_BUCKET:latest,redis-url=redis-url:latest,OPENAI_API_KEY=OPENAI_API_KEY:latest \
    --region=us-central1 \
    --project=pcpeprod
    gcloud run services update-traffic multi-model-doc-chat --to-latest --region=us-central1
    

    gcloud config set account ${cdc_oman_st} && gcloud config set project cdc-oman-st
    docker push gcr.io/cdc-oman-st/multi-model-doc-chat:$RELEASE_VERSION
    gcloud run deploy multi-model-doc-chat \
    --image=gcr.io/cdc-oman-st/multi-model-doc-chat:$RELEASE_VERSION \
    --allow-unauthenticated \
    --port=8000 \
    --service-account=196923721334-compute@developer.gserviceaccount.com \
    --min-instances=1 \
    --set-env-vars= \
    --set-cloudsql-instances=cdc-oman-st:us-central1:disearch-db \
    --vpc-connector=projects/cdc-oman-st/locations/us-central1/connectors/disearch-vpc-connector \
    --vpc-egress=private-ranges-only \
    --set-secrets=DB_HOST=DB_HOST:latest,DB_USER=DB_USER:latest,DB_PASSWORD=DB_PASSWORD:latest,DB_SCHEMA=DB_SCHEMA:latest,vertexai-referer=vertexai-referer:latest,MULTI_MODEL_GCP_BUCKET=MULTI_MODEL_GCP_BUCKET:latest,redis-url=redis-url:latest,OPENAI_API_KEY=OPENAI_API_KEY:latest \
    --region=us-central1 \
    --project=cdc-oman-st
    gcloud run services update-traffic multi-model-doc-chat --to-latest --region=us-central1


    gcloud config set account ${disearch_vertexai} && gcloud config set project disearch-vertexai
    docker push gcr.io/disearch-vertexai/multi-model-doc-chat:$RELEASE_VERSION
    gcloud config set project disearch-vertexai
    gcloud run deploy multi-model-doc-chat \
    --image=gcr.io/disearch-vertexai/multi-model-doc-chat:$RELEASE_VERSION \
    --allow-unauthenticated \
    --port=8000 \
    --service-account=714482271007-compute@developer.gserviceaccount.com \
    --min-instances=1 \
    --set-env-vars= \
    --set-cloudsql-instances=disearch-vertexai:us-central1:disearch-db \
    --vpc-connector=projects/disearch-vertexai/locations/us-central1/connectors/disearch-vpc-connector \
    --vpc-egress=private-ranges-only \
    --set-secrets=DB_HOST=DB_HOST:latest,DB_USER=DB_USER:latest,DB_PASSWORD=DB_PASSWORD:latest,DB_SCHEMA=DB_SCHEMA:latest,vertexai-referer=vertexai-referer:latest,MULTI_MODEL_GCP_BUCKET=MULTI_MODEL_GCP_BUCK…

    #script      

    #!/bin/bash
      
      # Define the list of images
      images=(
          "gcr.io/disearch/doc_chat:latest"
          "gcr.io/disearch/file-upload-pubsub:latest"
          "gcr.io/disearch/pdf-convert-pubsub:latest"
          "gcr.io/disearch/metadata-pubsub:latest"
          "gcr.io/disearch/vertex-ai-followup"
          "gcr.io/disearch/vertex-ai-summary"
          "gcr.io/disearch/vertexai-citation"
          "gcr.io/disearch/vertexai"
      )
      
      # Define the list of target projects
      projects=(
          "gcr.io/diseracharetecprod"
          "gcr.io/disearch-vertexai"
          "gcr.io/pcpeprod"
      )
      
      # Loop through each image and project to pull and push
      for image in "${images[@]}"; do
          # Pull the image from disearch
          echo "Pulling image: $image"
          docker pull "$image"
      
          # Get image name without the project prefix
          image_name=$(echo "$image" | cut -d'/' -f 3-)
      
          # Push to each project
          for project in "${projects[@]}"; do
              target_image="$project/$image_name"
              echo "Tagging and pushing image to: $target_image"
              
              # Tag the image for the target project
              docker tag "$image" "$target_image"
      
              # Push the tagged image
              docker push "$target_image"
          done
      done


      ## command for service account
      echo "Image pull and push process completed."
      gcloud config set account ${disearch} && gcloud config set project disearch
      gcloud run services list
      gcloud config set account ${pcpeprod} && gcloud config set project pcpeprod
      gcloud run services list
      gcloud config set account ${disearchmt_dev} && gcloud config set project disearchmt-dev
      gcloud run services list

## script  for creating the service account which help you to authenticate the gcp cloud from jenkins or any other server
      
      #!/bin/bash
      set -e
      # List of projects
      projects=(
          "disearch"
          "disearchmt-dev"
          "pcpeprod"
          "cdc-oman-st"
          "disearch-vertexai"
          "diseracharetecprod"
          "lumos-disearch-st"
          "world-learning-400909"
      )
      
      # Service account name and display name
      SERVICE_ACCOUNT_NAME="jenkins"
      DISPLAY_NAME="Jenkins Service Account"
      ROLES=(
          "roles/artifactregistry.repoAdmin"      # Artifact Registry Create-on-Push Repository Administrator
          "roles/run.developer"                   # Cloud Run Developer
          "roles/run.builder"                     # Delete, Download & Upload Artifacts
          "roles/container.serviceAgent"          # Kubernetes Engine Service Agent
          "roles/cloudfunctions.developer"        # Cloud Functions Developer
          "roles/pubsub.editor"                   # Pub/Sub Editor Update & Create Pub/Sub Topics & Subscription
          "roles/cloudsql.viewer"                 # Provides read-only access to Cloud SQL resources.
          "roles/storage.objectViewer"            # Grants access to view objects and their metadata, excluding ACLs. Can also list the objects in a bucket.  
          "roles/secretmanager.secretAccessor"     # Secret Manager Secret Accessor
          "roles/secretmanager.secretVersionManager"  # Secret Manager Secret Version Manager
          "roles/secretmanager.viewer"            # To list down secrets
      )
      
      # Directory to save service account keys
      KEY_DIR="./service-account-keys"
      mkdir -p $KEY_DIR
      
      # Create the service account, assign roles, and generate keys for each project
      for project in "${projects[@]}"; do
          echo "Processing project: $project"
      
          # Create the service account
          gcloud iam service-accounts create $SERVICE_ACCOUNT_NAME \
              --display-name="$DISPLAY_NAME" \
              --project=$project
      
          # Assign roles to the service account
          for role in "${ROLES[@]}"; do
              gcloud projects add-iam-policy-binding $project \
                  --member="serviceAccount:$SERVICE_ACCOUNT_NAME@$project.iam.gserviceaccount.com" \
                  --role="$role"
          done
      
          # Generate and download the service account key
          gcloud iam service-accounts keys create "$KEY_DIR/$project.json" \
              --iam-account="$SERVICE_ACCOUNT_NAME@$project.iam.gserviceaccount.com" \
              --project=$project
      
          echo "Service account, roles, and key file created for project: $project"
      done
      
      echo "All projects processed successfully. Keys are saved in $KEY_DIR."


    after that, use below command for authenticate the service account using key from jenkins server, for this you need to download key first from gcp  and save keys in jenkins server file.. after that use below command

    gcloud auth activate-service-account my-service-account@my-project.iam.gserviceaccount.com --key-file=/path/to/key-file.json

    gcloud config set account ${disearch} && gcloud config set project disearch   ---> we have create global variable in jenkins and give our service account name their.. 

    once  done, your jenkins server get access of gcp cloud using service account..


## How to determine whether a subnet is public or private, 

    you can evaluate its configuration based on routing and accessibility. Here's a step-by-step guide:
    
    1. Check the Subnet's Route Table
    Public Subnet: The route table associated with the subnet includes a route that directs traffic to the internet gateway.
    Example: 0.0.0.0/0 → Internet Gateway (igw-xxxxx).
    Private Subnet: The route table does not include a route to the internet gateway. Instead, it may route traffic to a NAT gateway or NAT instance for outbound internet access.
    
    2- Determine Access to the Internet
    Public Subnet: Resources in this subnet can have direct access to the internet (e.g., EC2 instances with public IPs can send and receive traffic to/from the internet).
    Private Subnet: Resources cannot directly access the internet without a NAT gateway or instance.

    public subnet per internet sa traffic ati ha,or subnet sa traffic internet per wapis b jati ha, but private subnet per internet sa traffic ni asakhti, but private subnet ki internet per traffic ja sakhti ha using NAT gatway attached to with private subnet, jo k private subnet ki traffic ko subnet per send kry ga, or phir waha sa traffic route table sa hoty howy internet per jarhi hogi.. 

## Difference b/w application and network loadbalancer

      Application Load Balancer (ALB) and Network Load Balancer (NLB) operate on different layers of the OSI model:
      
      Application Load Balancer (ALB)
      Layer: Layer 7 (Application Layer)
      Details:
      Operates at the HTTP/HTTPS level.
      Designed for content-based routing (e.g., routes requests based on hostnames, paths, headers, or query strings).
      Supports advanced features like WebSocket, TLS termination, and redirects.
      Ideal for applications that rely on HTTP protocols, such as REST APIs, websites, or web apps.
      

      Network Load Balancer (NLB)
      Layer: Layer 4 (Transport Layer)
      Details:
      Operates at the TCP/UDP level.
      Focused on connection-based routing (e.g., routes traffic based on IP addresses and ports).
      Provides ultra-low latency and can handle millions of requests per second.
      Suitable for applications that require high performance and low latency, such as gaming, streaming, or non-HTTP applications.
      

      Key Differences
      Feature	Application Load Balancer (ALB)	Network Load Balancer (NLB)
      OSI Layer	Layer 7	Layer 4
      Protocol Support	HTTP, HTTPS	TCP, UDP
      Routing	Content-based (host, path, etc.)	Connection-based (IP, port)
      Latency	Slightly higher	Extremely low
      
      Use Case	Web applications, APIs	High-performance, low-latency apps
      If you’re deploying an application, the choice between ALB and NLB depends on whether your application needs Layer 7 features like path-based routing or simpler Layer 4 features like load balancing for non-HTTP traffic. Let me know if you want to discuss further!

## Can we create project without having organization in GCP

      Yes, you can create a project in Google Cloud Platform (GCP) without creating an organization. This is commonly done when you're using a personal Google account instead of a Google Workspace or Cloud Identity account. Here's how it works:
      
      When You Don't Have an Organization
      Project Ownership: The project will be associated with your personal Google account.
      No Organization Policies: You won’t have access to organization-level policies or controls, such as central IAM or resource hierarchy.
      Billing: You’ll still need to link a billing account to the project, which can also be personal.
      Steps to Create a Project Without an Organization
      Log in to GCP Console:
      
      Visit Google Cloud Console.
      Go to Project Selector:
      
      In the top navigation bar, click the Project Selector dropdown.
      Click New Project.
      Enter Project Details:
      
      Project Name: Enter a unique name.
      Location: Leave it as "No organization" if you don’t have one.
      Billing Account: Assign a billing account (you can use the free trial if you don't have one).
      Click Create:
      
      Your project will be created and associated with your personal Google account.
      Limitations Without an Organization
      No Centralized Resource Management:
      
      You can’t use folders or enforce organization-wide policies.
      Account Tied to Project:
      
      If you lose access to your personal account, you may lose access to the project.
      IAM Complexity:
      
      Role and permission assignments are managed directly at the project level.
      Best Practices Without an Organization
      Enable Billing Alerts:
      Ensure your billing account has budget alerts to prevent unexpected costs.
      Set Up IAM Roles:
      Assign roles carefully to ensure proper access control.
      Consider Upgrading:
      If you're scaling, consider setting up a Google Workspace or Cloud Identity to create an organization for better resource management.
            
      
## how to create organization and project inside the orgainzation
      
      How to create organization in gcp:
      ----------------------------------
      
      for organization you need to have
      
      1- domain          --> purchase domain from "domains.google.com" > click "get new domain" > search domain is available or not.> complete payment process
      2- workspace account
      
      once purchase the domain, now complete the google workspace account sets.
      
      **you need to have google workspace account**
      
      browse the "google workspace" on google and start completing workspace process.
      
      like give:
      
      name:
      email:
      workspace will be used:
      i will setup workspace with specfic domain:   give domain name here     ---> which you have just purchased
      
      after this it will switch the page to workspace account.
      
      now here create user for your organization: click on **add user** and give information
      
      like:
      name: 
      surname:
      primary email:
      secondary email:
      phone:
      
      
      now login organization with new user using, brows and open "console.cloud.google.com" and you would see your created organization will pop up in gcp organization section.
      
      
      now create project inside the organization
      -------------------------------------------
      
      remember: you can also create project without creating the organization 
      
      Create a Project
      Projects are the primary way to organize resources in GCP. They belong to an organization (if one exists) or to a user account.
      
      Steps to Create a Project:
      Log in to Google Cloud Console:
      
      Go to Google Cloud Console.
      Navigate to Projects:
      
      Click on the Project Selector dropdown (top menu bar).
      Click New Project.
      Enter Project Details:
      
      Project Name: Choose a unique name for your project.
      Organization: If you have an organization, select it here. If not, the project will belong to your Google account.
      Billing Account: Assign a billing account. If you don’t have one, set up a free trial or create a billing account in the Billing section of the console.
      Create the Project:
      
      remember: if you want to get control of organization level, do this

      - create organization by following above mention steps
      - once you successfully create organization, then create folder inside the organization. 
      - now in that folder you can create multiple projects. 

      now when you apply some permission at organization level using that folders, the permission will autmatically apply on all projects inside the folder.

     you can also create projects under organization without creating folders and you can also set permission at organization level using one of the projects under this organization, but it will only apply on that organization project only but if you want to apply organization permission on multiple project than you should use folder,


## How to create users in aws under IAM

    steps to create user
    --------------------

    IAM > user > adduser > username > click on "provide user access to aws management console"> click on "i want to create iam user"> generate custom/autoassign password >next>click on  "attach direct policy to user" > click on "administrator access" > next >review this >create user
    
    
    then get user credentails
    -------------------------
    
    - console url for login
    - username
    - password
    create user in aws

## AWS Databases: Structured and Unstructured
      
    AWS offers both structured and unstructured database solutions to meet various data storage and management needs. Here's a detailed breakdown of what these databases are, their examples, and their key differences:

    Structured Databases
    Definition: Databases designed to store highly organized, relational data that follows a schema (e.g., tables with rows and columns).
    Characteristics:
    
    Data is stored in predefined formats (schemas).
    Data is easy to query using SQL.
    Ensures strong consistency and supports transactions.
    
    AWS Examples of Structured Databases:
    
    Amazon RDS (Relational Database Service):
    Fully managed service supporting multiple database engines like MySQL, PostgreSQL, Oracle, and SQL Server.
    
    Amazon Aurora:
    A high-performance, fully managed relational database compatible with MySQL and PostgreSQL.
    
    Amazon Redshift:
    A fully managed data warehouse optimized for analyzing structured data.
    
    Unstructured Databases
    Definition: Databases designed to handle data that does not fit into a traditional table structure or lacks a specific schema (e.g., documents, images, videos, logs).
    Characteristics:
    Flexible schema or schema-less design.
    Data can be in various formats like JSON, BSON, XML, or binary.
    Often optimized for scalability and horizontal growth.
    AWS Examples of Unstructured Databases:
    
    Amazon DynamoDB:
    A fully managed NoSQL database for key-value and document data.
    
    Amazon S3 (Simple Storage Service):
    Object storage service for storing unstructured data like files, images, videos, and backups.
    
    Amazon ElastiCache:
    A caching service (supports Redis and Memcached) often used for unstructured, ephemeral data.
    
    Amazon DocumentDB:
    A managed NoSQL document database that is MongoDB-compatible.
    
    Amazon Neptune:
    A graph database for highly connected, unstructured data like social networks or recommendation systems.
    Key Differences Between Structured and Unstructured Databases
    Feature	Structured Database	Unstructured Database
    Schema	Fixed schema (rigid structure).	Flexible or schema-less.
    Data Format	Tabular (rows and columns).	Documents, key-value pairs, objects, binary.
    Query Language	SQL (Structured Query Language).	NoSQL, custom APIs, or query methods.
    Data Types	Numbers, strings, dates.	JSON, BSON, XML, multimedia files.
    Use Cases	Financial systems, ERP, CRM.	IoT data, content management, real-time analytics.
    Consistency	Strong consistency.	Eventual or tunable consistency.
    Scalability	Vertical (increase machine power).	Horizontal (add more machines).
    
    When to Use Which?
    Requirement	Recommended Database
    Need a relational model and ACID compliance	Amazon RDS, Amazon Aurora
    Need to store hierarchical or JSON-like data	Amazon DynamoDB, Amazon DocumentDB
    Need to store large objects (files, images)	Amazon S3
    Need to process and analyze structured data	Amazon Redshift
    Need to handle real-time or high-speed caching	Amazon ElastiCache
    Need to handle highly connected data	Amazon Neptune
     
## Structured data
   
   Structured data is particularly useful when you’re dealing with discrete, numeric data. **Examples of this type of data include financial operations, sales and marketing figures, and scientific modeling. You can also use structured data in any case where records with multiple, short-entry text, numeric, and enumerated fields are required, such as HR records, inventory listings, and housing data.**
    
## Unstructured data
   
   Unstructured data is used when a record is required and the data won’t fit into a structured data format. **Examples include video monitoring, company documents, and social media posts. You can also use unstructured data where it isn’t efficient to store the data in a structured format, such as Internet of Things (IoT) sensor data, computer system logs, and chat transcripts.**

## Aws structual and unstructural database

        Here’s a list of AWS databases categorized as structured and unstructured based on their architecture and data handling capabilities:
        
        Structured Databases (Relational Databases)
        These databases store highly organized, relational data with a fixed schema (tables, rows, columns).
        
        AWS Services
        Amazon RDS (Relational Database Service):
        
        Supports multiple relational database engines: MySQL, PostgreSQL, Oracle, SQL Server, MariaDB, and Amazon Aurora.
        Fully managed service with automated backups, patching, and scaling.
        Amazon Aurora:
        
        High-performance, fully managed relational database.
        Compatible with MySQL and PostgreSQL.
        
        Amazon Redshift:
        
        Fully managed data warehouse for structured data analytics.
        Optimized for complex queries and Business Intelligence (BI) applications.
        
        Unstructured Databases (NoSQL, Object, and Other Non-Relational Databases)
        These databases handle flexible or schema-less data types, including documents, key-value pairs, and objects.
        
        AWS Services
        Amazon DynamoDB:
        
        Fully managed NoSQL database for key-value and document data.
        Highly scalable with low-latency performance.
        Amazon S3 (Simple Storage Service):
        
        Object storage service for unstructured data like images, videos, files, and backups.
        Supports a variety of data types and scales automatically.
        Amazon ElastiCache:
        
        In-memory data store supporting Redis and Memcached.
        Optimized for caching and fast, ephemeral data access.
        Amazon DocumentDB:
        
        Managed NoSQL document database compatible with MongoDB.
        Designed for JSON-based workloads.
        Amazon Neptune:
        
        Graph database for highly connected, unstructured data like social networks or recommendation engines.
        Amazon Timestream:
        
        Managed time-series database optimized for time-stamped data (IoT applications, logs, etc.).
        Amazon Keyspaces (for Apache Cassandra):
        
        Managed NoSQL database service for wide-column data models.
        Compatible with Apache Cassandra.
        AWS Lake Formation:
        
        Used to build and manage data lakes for storing large amounts of unstructured or semi-structured data.
        Comparison: Structured vs. Unstructured Databases in AWS
        Feature	Structured Databases (Relational)	Unstructured Databases (NoSQL, Object)
        Schema	Fixed schema (rigid structure).	Flexible or schema-less.
        Data Types	Tabular (rows and columns).	Documents, objects, key-value pairs, binary.
        Query Language	SQL (Structured Query Language).	NoSQL, APIs, or custom queries.
        Consistency	Strong consistency, ACID compliance.	Eventual or tunable consistency.
        Scalability	Vertical scaling.	Horizontal scaling (add machines).
        Use Cases	Financial systems, ERP, CRM.	IoT, content management, real-time analytics.
        Let me know if you need help exploring or configuring any of these services!
        
## How to route traffic b/w regions

      Routing traffic across clusters in two different regions using a single DNS involves deploying a global load balancing solution that dynamically directs traffic based on factors like geographic proximity, availability, or resource usage. Here’s how you can achieve this:
      
      1. Use a Global Load Balancer
      Cloud providers like AWS, Google Cloud, and Azure offer global load balancers designed for multi-region traffic routing.
      
      Google Cloud (GCP) Global Load Balancer
      Setup:
      
      Use a global HTTP(S) Load Balancer.
      Deploy backend services pointing to your Kubernetes clusters in different regions.
      Configure the load balancer to route traffic based on geographic proximity or latency.
      DNS Integration:
      
      Assign a custom domain to the load balancer.
      The load balancer will resolve the DNS and route traffic accordingly.
      Steps in GCP:
      
      Expose each cluster service using a regional NEG (Network Endpoint Group).
      Attach NEGs as backends to the global load balancer.
      Add DNS records to map the domain to the global load balancer.
     
      2- **AWS Global Accelerator**
      Setup:
      
      Use AWS Global Accelerator to route traffic to regional endpoints.
      Deploy an ALB (Application Load Balancer) in each region to handle cluster traffic.
      DNS Integration:
      
      AWS provides a static global IP for Global Accelerator.
      Update your DNS (e.g., Route53) to point to this global IP.
      
      Traffic Routing:
      
      Global Accelerator routes traffic based on health checks, latency, or geo-location.
      
      2. **Use an External DNS and Traffic Management Tool**
      DNS Providers with Geo-Routing
      Providers like **Cloudflare**, Akamai, or **AWS Route 53 can handle geographic routing by resolving your DNS requests to the nearest cluster.**
      Steps:
      
      Deploy your application in both clusters.
      Use a DNS provider to create a single domain record.
      Configure routing policies:
      Geo-routing: Route traffic based on the user’s geographic location.
      Latency-based routing: Route traffic to the region with the lowest latency.
      Example: Route 53 Latency-Based Routing
      Create a hosted zone in Route 53.
      Set up latency-based routing records pointing to:
      Load balancer in Region 1.
      Load balancer in Region 2.
      Route 53 dynamically resolves DNS queries to the region with the lowest latency.
     
     3. Using Service Mesh with Multi-Cluster Setup
      Istio Multi-Cluster with External DNS
      Deploy Istio in Both Clusters:
      
      Use Istio’s multi-cluster capabilities to connect services in both regions.
      Choose between:
      Primary-Primary: Both clusters handle traffic independently.
      Primary-Remote: One cluster handles control-plane operations.
      Single DNS:
      
      Expose Istio’s ingress gateway in both clusters with the same domain.
      Use a global load balancer (or geo-aware DNS) to route traffic.
      Failover:
      
      Configure Istio for cross-cluster failover if one region becomes unavailable.

## what is /dev

    The /dev directory in Ubuntu contains device files that represent hardware and software interfaces, such as storage, input/output devices, and system utilities. It acts as the communication point between the system and hardware components

## Building image from local dockerfile and pushing it to gcr

      gcloud builds submit --tag gcr.io/world-learning-400909/disearch:latest --timeout=9000 --machine-type=e2-highcpu-32

## what is the difference b/w resource and module in terraform
    
    The difference between a Terraform resource and a Terraform module is as follows:
    
    Terraform Resource:
    A resource is a single, individual infrastructure component that Terraform manages. It could represent an AWS EC2 instance, an S3 bucket, a database, etc.
    Example:
    hcl
    Copy code
    resource "aws_instance" "example" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t2.micro"
    }
    Terraform Module:
    A module is a collection of resources that are grouped together to define a reusable unit of infrastructure. Modules can contain resources, output values, input variables, and other modules.
    Modules allow you to organize and reuse infrastructure code across multiple configurations.
    You can create custom modules or use public ones from the Terraform Registry.
    Example:
    hcl
    Copy code
    module "network" {
      source = "./modules/network"
      cidr_block = "10.0.0.0/16"
    }
    Key Differences:
    Scope: A resource is a single unit of infrastructure, while a module is a group of resources.
    Reusability: Modules are designed to be reusable, while resources are typically specific to one configuration.
    Organization: Modules help organize complex configurations into smaller, more manageable pieces.
    In summary, resources are the individual components that Terraform manages, while modules are collections of resources that allow for better organization and reuse of infrastructure code
