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
      
      


      
      
      
      
      
      
      
