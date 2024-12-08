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

## What is the difference b/w resource and module in terraform
    
    The difference between a Terraform resource and a Terraform module is as follows:
    
    **Terraform Resource:**
    A resource is a single, individual infrastructure component that Terraform manages. It could represent an AWS EC2 instance, an S3 bucket, a database, etc.
    Example:
    hcl
    Copy code
    resource "aws_instance" "example" {
      ami           = "ami-0c55b159cbfafe1f0"
      instance_type = "t2.micro"
    }
    
    **Terraform Module:**
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

## What is the difference b/w terraform reconfigure and migration

    The terms Terraform reconfigure and Terraform migration are often used in different contexts but have distinct meanings in Terraform workflows:
    
    **Terraform Reconfigure:**
    Reconfigure refers to modifying your Terraform configuration to make adjustments based on changes in your infrastructure needs or requirements.
    
    This usually involves changing resource definitions, adding new resources, or altering input variables in your .tf files.
    
    After a reconfiguration, you typically run:
    
    terraform plan
    terraform apply
    to see the changes Terraform will make to your infrastructure and to apply them.
    
    Use Case: Reconfiguring is done when you want to change the setup of your infrastructure, like adding a new instance, resizing a resource, or changing variables.
    
    **Terraform Migration:**
    Migration refers to moving your Terraform-managed infrastructure from one state to another, often due to significant changes in infrastructure providers, backends, or moving to a different version of Terraform or modules.
    
    This can involve:
    
    Migrating resources between different cloud providers (e.g., AWS to Azure).
    Moving state files between different backends (e.g., from local state to remote state like S3 or Terraform Cloud).
    Upgrading or changing Terraform versions, modules, or providers.
    Use Case: Migrations are necessary when moving or upgrading an existing infrastructure setup, especially if it involves changes to state files, provider configurations, or backend configurations.
    
    Key Differences:
    Reconfigure is about changing resource definitions or variables in your Terraform configuration to adjust your infrastructure.
    Migration typically involves moving or upgrading infrastructure or state, often across different platforms, versions, or configurations.
    In summary, reconfiguration is for making direct changes to the infrastructure setup, while migration involves moving or upgrading your infrastructure or state to a new     context.
    
    
## What is terraform backend and why not we use github for terraform backend instead of cloud bucket

    What is a Terraform Backend?
    
    A Terraform backend is the location where Terraform stores its state files. The state file contains the current configuration of your infrastructure and is used by Terraform to track changes. The backend determines how the state is stored, accessed, and shared across multiple users or systems.
    
    Backends can be:
    
    Local Backend: State is stored on your local filesystem.
    Remote Backend: State is stored in a remote location, like AWS S3, Google Cloud Storage, or HashiCorp Consul.
    Remote backends are commonly used for collaboration, state sharing, and ensuring consistency in infrastructure management.
    
    Why Not Use GitHub for Terraform Backend Instead of a Cloud Bucket?
    While you might consider storing Terraform state files in a Git repository like GitHub, there are significant reasons why this is not recommended:
    
    1. Concurrency and Locking Issues:
    
    Terraform relies on its state file to make decisions about the current state of the infrastructure. When multiple people or processes run terraform apply simultaneously, it can result in conflicting changes or corrupted state.
    Cloud-based backends (like AWS S3 with DynamoDB for locking) provide state locking to ensure only one process can update the state at a time, preventing race conditions and conflicts. GitHub does not provide any native support for locking.
    
    2. File Integrity:
    
    Terraform state files are binary JSON files and can grow large depending on the size of your infrastructure. GitHub is designed for text-based files, and it is not optimized for storing large binary files. This can lead to slow performance, large repository sizes, and potential loss of data integrity.
    Remote backends like S3 or Azure Blob Storage are optimized for handling state files, supporting efficient read/write operations.
    
    3. Versioning and History:
    
    GitHub can track changes to files through version history, but it does not have the ability to manage complex Terraform state changes effectively. If a state file is updated and pushed to GitHub by multiple users, merging those changes can result in broken states or conflicting resources.
    Remote backends handle versioning and history through built-in mechanisms, ensuring that Terraform always has access to the most recent state file without the risk of accidental overwriting or corruption.
    
    4. Collaboration:
    
    Using GitHub as a backend for state can complicate collaboration. Since Terraform state files are updated frequently, managing these changes within GitHub would require manual commits and pushes, making it harder to track changes and collaborate in real-time.
    Remote backends are designed for collaboration and can be accessed by multiple users concurrently, ensuring that Terraform operations are consistent across teams.
    
    5. Security:
    
    Terraform state files can contain sensitive information, such as credentials, secret keys, and passwords. Storing these files in GitHub (especially in a public repository) can expose that sensitive data.
    Cloud storage backends (like AWS S3, GCP Cloud Storage, etc.) allow for secure access control through IAM roles, encryption at rest, and fine-grained access policies, which help protect sensitive data.
    
    6. State Management Features:
    
    Cloud backends typically offer features like state encryption, automatic state locking, versioning, and remote access which are specifically designed for infrastructure management. These features are vital for ensuring the integrity and safety of the Terraform state, especially in teams or automated workflows.
    Conclusion:
    Using GitHub to store Terraform state files is not ideal because it lacks the features needed for safe, concurrent, and efficient state management (such as locking, versioning, and secure access). Cloud-based storage solutions (like AWS S3, Google Cloud Storage, or Terraform Cloud) are designed specifically for managing Terraform state files and provide the necessary features for secure and scalable infrastructure management.

## What to do on avoid displaying sensitive information like the RDS ARN or password in your Terraform plan or output,

    If you want to avoid displaying sensitive information like the RDS ARN or password in your Terraform plan or output, you can take several approaches to securely manage and hide sensitive values.
    
    1. Mark the Variables as Sensitive
    You can mark variables or outputs as sensitive in Terraform to prevent them from being shown in the plan or output. This helps avoid exposing sensitive data, such as passwords or ARNs, in the Terraform CLI output.
    
    Example for Variables:
    When defining your variable for RDS password or ARN, you can mark it as sensitive like this:
    
    hcl
    Copy code
    variable "rds_password" {
      description = "The password for the RDS instance"
      type        = string
      sensitive   = true
    }
    
    variable "rds_arn" {
      description = "The ARN for the RDS instance"
      type        = string
      sensitive   = true
    }
    Example for Outputs:
    When defining outputs that you don't want to display in the Terraform plan or apply output, you can mark them as sensitive:
    
    hcl
    Copy code
    output "rds_password" {
      value     = aws_db_instance.example.password
      sensitive = true
    }
    
    output "rds_arn" {
      value     = aws_db_instance.example.arn
      sensitive = true
    }
    With sensitive = true, Terraform will not display these values in the CLI output, even though they will still be stored in the Terraform state.
    

## Is it possible to implement the same Terraform manifest (configuration) on multiple clouds,

    Yes, it is possible to implement the same Terraform manifest (configuration) on multiple clouds, but it requires careful handling of cloud-specific resources, providers, and configurations.
    
    Key Considerations for Using Terraform Across Multiple Clouds:
    Use Multiple Providers: Terraform allows you to define multiple providers in your configuration. Each cloud (e.g., AWS, Azure, Google Cloud, etc.) will have its own provider block that specifies how Terraform interacts with that cloud.
    
    Example: Using AWS and Azure providers in the same configuration:
    
    hcl
    Copy code
    # Define the AWS provider
    provider "aws" {
      region = "us-west-2"
    }
    
    # Define the Azure provider
    provider "azurerm" {
      features {}
    }
    
    # AWS resources
    resource "aws_s3_bucket" "my_bucket" {
      bucket = "my-aws-bucket"
      acl    = "private"
    }
    
    # Azure resources
    resource "azurerm_resource_group" "my_group" {
      name     = "my-azure-resource-group"
      location = "East US"
    }
    Using Provider Aliases: When you need to use multiple accounts or regions within the same provider (e.g., AWS), you can use provider aliases to differentiate between them.
    
    Example: Using multiple AWS regions with aliases:
    
    hcl
    Copy code
    provider "aws" {
      region = "us-west-2"
      alias  = "us_west"
    }
    
    provider "aws" {
      region = "us-east-1"
      alias  = "us_east"
    }
    
    # Resource in the 'us-west-2' region
    resource "aws_s3_bucket" "my_bucket_west" {
      provider = aws.us_west
      bucket   = "my-west-bucket"
    }
    
    # Resource in the 'us-east-1' region
    resource "aws_s3_bucket" "my_bucket_east" {
      provider = aws.us_east
      bucket   = "my-east-bucket"
    }
    Conditional Logic for Cloud-Specific Resources: Some resources are specific to a cloud provider, so you'll need to conditionally create resources based on the provider or use cloud-specific variables.
    
    Example: Cloud-agnostic resource with conditionals:
    
    hcl
    Copy code
    variable "cloud_provider" {
      type    = string
      default = "aws"  # Change to "azure" or "google" as needed
    }
    
    resource "aws_s3_bucket" "example" {
      count  = var.cloud_provider == "aws" ? 1 : 0
      bucket = "my-aws-bucket"
    }
    
    resource "azurerm_storage_account" "example" {
      count                        = var.cloud_provider == "azure" ? 1 : 0
      name                         = "myazurstorageacct"
      resource_group_name          = azurerm_resource_group.example.name
      location                     = "East US"
      account_tier                  = "Standard"
      account_replication_type    = "LRS"
    }
    Cloud-Agnostic Infrastructure: To maximize reusability, you can abstract your configuration using modules and make your infrastructure as cloud-agnostic as possible. However, you may need to customize parts of the configuration to handle cloud-specific resources (like storage, networking, etc.).
    
    For example, you can create a module to provision resources, and within that module, use conditionals to decide which provider-specific resources to deploy based on input variables.
    
    State Management: When using multiple clouds, managing state becomes important. You may want to use remote backends (e.g., S3, Terraform Cloud) to store the state centrally. Ensure that the state file does not become overly complex when managing resources across multiple clouds.
    
    Example: Using S3 as a backend:
    
    hcl
    Copy code
    terraform {
      backend "s3" {
        bucket = "my-terraform-state"
        key    = "infrastructure/terraform.tfstate"
        region = "us-west-2"
      }
    }
    Module Reusability: Consider using Terraform modules to abstract out shared infrastructure patterns and configurations. You can create separate modules for each cloud provider and then choose which one to use in your main configuration.
    
    Example: A simple module for AWS resources:
    
    hcl
    Copy code
    # AWS Module - modules/aws/s3_bucket/main.tf
    resource "aws_s3_bucket" "my_bucket" {
      bucket = "my-aws-bucket"
      acl    = "private"
    }
    Then, you can use this module conditionally or multiple times in your main Terraform configuration.
    
    Multi-Cloud Management Tools: Tools like Terraform Cloud or Terraform Enterprise can help you manage multiple cloud providers and streamline the process of provisioning infrastructure across different clouds in a centralized manner.
    
    Challenges and Best Practices:
    Provider Compatibility: Some features are specific to a provider, so you might not be able to fully replicate a resource from one cloud to another.
    State File Management: Be careful with the state file, especially when dealing with multiple clouds and environments. Consider using Terraform Cloud or remote backends like AWS S3 to manage your state file.
    Resource Dependencies: If you have resources that need to interact across clouds (e.g., VPC peering between AWS and Azure), managing cross-cloud dependencies can be more complex.
    Conclusion:
    Yes, you can implement the same Terraform manifest across multiple clouds by using multiple providers, cloud-specific resources, and modules. However, you must account for differences in resources and services between providers and handle them accordingly in your Terraform configuration. Using provider aliases, conditionals, and remote backends will help you manage a multi-cloud infrastructure setup effectively.
    
##  If you have a Kubernetes cluster with 3 nodes in 3 different regions and want to manage where specific pods are scheduled to run in a particular region, 
    
    If you have a Kubernetes cluster with 3 nodes in 3 different regions and want to manage where specific pods are scheduled to run in a particular region, you can achieve this by using affinity and taints and tolerations in Kubernetes. These mechanisms allow you to control pod placement across nodes located in different regions.
    
    Here's how you can manage pod placement on specific regions:
    1. Use Node Affinity
    Node affinity is a concept in Kubernetes that allows you to constrain which nodes your pods are eligible to be scheduled based on labels on nodes. You can label your nodes based on their region, and then use affinity rules to make sure the pods are scheduled on nodes in specific regions.
    
    Step-by-step guide to configure Node Affinity:
    Label your nodes based on their region: You need to label each node in your cluster with the region information. For example, if your nodes are in us-east-1, us-west-2, and eu-central-1, you can label the nodes as follows:
    
    bash
    Copy code
    kubectl label nodes <node-name> region=us-east-1
    kubectl label nodes <node-name> region=us-west-2
    kubectl label nodes <node-name> region=eu-central-1
    Define affinity in your Pod specification: Use the affinity field in your pod specification to schedule the pod to a specific region. Here's an example pod definition with node affinity to ensure the pod runs in the us-west-2 region:
    
    yaml
    Copy code
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: "region"
                    operator: In
                    values:
                      - "us-west-2"
      containers:
        - name: my-container
          image: my-image
    In this example:
    
    The matchExpressions specifies that the pod should only be scheduled on nodes with the label region=us-west-2.
    2. Use Taints and Tolerations
    Taints and tolerations allow nodes to repel pods unless those pods explicitly tolerate the taint. This is another method to ensure that certain types of workloads only run on nodes in specific regions.
    
    Step-by-step guide to configure Taints and Tolerations:
    Taint the nodes with the region information: Taint your nodes to indicate the region they belong to. This will "repel" any pod from being scheduled on those nodes unless the pod has the corresponding toleration.
    
    bash
    Copy code
    kubectl taint nodes <node-name> region=us-east-1:NoSchedule
    kubectl taint nodes <node-name> region=us-west-2:NoSchedule
    kubectl taint nodes <node-name> region=eu-central-1:NoSchedule
    Here, NoSchedule means that no pods will be scheduled on that node unless the pod tolerates the taint.
    
    Add tolerations in your pod specification: To allow your pods to run on specific nodes (regions), add tolerations in your pod spec. For example, to schedule a pod on the us-west-2 region:
    
    yaml
    Copy code
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      tolerations:
        - key: "region"
          operator: "Equal"
          value: "us-west-2"
          effect: "NoSchedule"
      containers:
        - name: my-container
          image: my-image
    In this example:
    
    The pod has a toleration for the region=us-west-2 taint, meaning it will be scheduled on nodes in the us-west-2 region.
    3. Pod Anti-Affinity (optional):
    If you want to ensure that pods from the same deployment or application are not scheduled on the same node (or even the same region), you can use pod anti-affinity. This can be useful if you want to distribute the pods across multiple regions.
    
    Example of Pod Anti-Affinity:
    yaml
    Copy code
    apiVersion: v1
    kind: Pod
    metadata:
      name: mypod
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                      - myapp
              topologyKey: "region"
      containers:
        - name: my-container
          image: my-image
    In this example:
    
    The podAntiAffinity ensures that pods with the label app=myapp are scheduled in different regions (topology key is region).
    4. Using Kubernetes Federation (Advanced):
    For more advanced scenarios where you want to manage Kubernetes clusters across multiple regions, you can use Kubernetes Federation to federate your clusters in different regions. Federation allows you to run a multi-cluster environment and manage workloads across these clusters.
    
    Federation Overview:
    Kubernetes Federation allows you to set up a control plane to manage multiple clusters across different regions and ensure your applications are resilient across these regions.
    Summary of Options:
    Node Affinity: Control where the pods run based on the node labels, like region=us-west-2.
    Taints and Tolerations: Ensure that only certain pods are scheduled on nodes with specific taints (e.g., region).
    Pod Anti-Affinity: Distribute pods across multiple regions (or nodes) to avoid co-locating them on the same region.
    Kubernetes Federation: For managing multiple clusters across different regions with a global control plane (more advanced).
    These methods allow you to fine-tune pod placement in multi-region clusters and optimize workloads according to your region-specific requirements.


What is the difference b/w liveness prob, readiness prob, startup prob.
        
        In Kubernetes, liveness, readiness, and startup probes are used to monitor the health and status of pods. These probes help Kubernetes determine when to restart, route traffic to, or consider a pod healthy based on specific criteria. Here's a breakdown of the differences between them:
        
        1. Liveness Probe
        Purpose: The liveness probe checks whether a pod is still running. If the probe fails, Kubernetes will restart the pod because it is considered unhealthy.
        Use case: Typically used when your application might enter a state from which it cannot recover (e.g., a deadlock or a critical failure).
        When to use: You should use a liveness probe if your application can become unhealthy and needs to be restarted to fix the problem.
        Example:
        yaml
        Copy code
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 5
        Error indication: If the liveness probe fails, Kubernetes will restart the pod. Common errors include:
        CrashLoopBackOff: The pod is repeatedly crashing.
        500 Internal Server Error: If your application is returning errors on the probe path.
        You would typically know it's a liveness probe issue if your pod is crashing frequently or stuck in a CrashLoopBackOff state.
        
        2. Readiness Probe
        Purpose: The readiness probe checks whether a pod is ready to accept traffic. If the probe fails, Kubernetes will stop sending traffic to the pod, but it will not restart it.
        Use case: Use readiness probes when your application needs time to warm up, initialize, or connect to external services before it can start receiving traffic (e.g., database connection setup).
        When to use: You should use a readiness probe if your app is dependent on external systems or has a startup time before it can handle traffic.
        Example:
        yaml
        Copy code
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
        Error indication: If the readiness probe fails:
        The pod won’t receive any traffic (load balancer, service, etc. will stop routing traffic to it).
        This could indicate the app isn't fully initialized or is dependent on an external service that isn't available yet.
        You know it’s a readiness issue if your app isn’t receiving traffic even though the pod is running without any restart problems.
        
        3. Startup Probe
        Purpose: The startup probe is used to determine if the application within the pod has started successfully. It’s especially useful for apps with a long initialization period (e.g., loading data or connecting to external services).
        Use case: Useful when your application requires longer startup times, and you want Kubernetes to wait for the app to start before performing the liveness or readiness checks.
        When to use: You should use a startup probe if your application has a long initialization phase, and you don’t want liveness or readiness probes to interfere with the start-up process.
        Example:
        yaml
        Copy code
        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        Error indication: If the startup probe fails:
        Kubernetes will consider the pod failed and restart it after the threshold is exceeded.
        Common issues would be delays in startup processes, such as slow database connection or large application initialization.
        You know it’s a startup issue if the pod starts but never gets beyond the startup phase, or Kubernetes keeps restarting the pod.
        
        Key Differences:
        Liveness Probe: Ensures the pod is alive; if failed, pod is restarted.
        Readiness Probe: Ensures the pod is ready to serve traffic; if failed, traffic is stopped, but no restart happens.
        Startup Probe: Ensures the application has started successfully; prevents premature liveness/readiness failures for apps with long start-up times.
        Error Examples for Indication:
        Liveness Probe:
        CrashLoopBackOff: The pod keeps failing and restarting.
        500 Internal Server Error: If the probe URL (e.g., /healthz) is failing due to an application error.
        Readiness Probe:
        Service not routing traffic: The pod is running, but it’s not receiving traffic because it failed the readiness probe.
        Startup Probe:
        Repeated restarts: The pod is continually restarting because it failed the startup probe.
        How to identify which probe is failing:
        Logs: Check pod logs (kubectl logs <pod-name>) to understand what is happening inside the container. You will often see errors related to failed health checks.
        Pod Status: Use kubectl describe pod <pod-name> to see the exact reason for failure. The output will show which probe failed (liveness, readiness, or startup) and the failure conditions.
        By understanding these probes and their error indications, you can better manage pod health and troubleshoot issues in your Kubernetes deployment
            
            
## What is the difference b/w replicaset and statefullset

        In Kubernetes, both ReplicaSets and StatefulSets are used to manage the deployment and scaling of pods, but they differ in their use cases and how they manage the pods. Here's a breakdown of the differences between ReplicaSet and StatefulSet:
        
        1. ReplicaSet
        Purpose: A ReplicaSet ensures that a specified number of identical pods are running at any given time. It is primarily used to scale and maintain stateless applications.
        Pod Identity: Pods managed by a ReplicaSet are interchangeable and have no persistent identity. If a pod is deleted, a new pod is created to replace it, and the new pod has a different name and IP address.
        Use Case: ReplicaSets are used for stateless applications, where the pods do not need to maintain a unique identity or persistent storage.
        Pod Naming: Pods created by a ReplicaSet are named randomly by Kubernetes with a prefix based on the ReplicaSet's name (e.g., myapp-abc123).
        Storage: Typically does not provide persistent storage or guarantees for storage across pods.
        Scaling: ReplicaSets handle scaling of the number of pods. They ensure that the specified number of pods is always running.
        Example: Web servers or stateless microservices.
        Example of a ReplicaSet YAML:
        yaml
        Copy code
        apiVersion: apps/v1
        kind: ReplicaSet
        metadata:
          name: my-replicaset
        spec:
          replicas: 3
          selector:
            matchLabels:
              app: myapp
          template:
            metadata:
              labels:
                app: myapp
            spec:
              containers:
              - name: myapp-container
                image: myapp-image
        2. StatefulSet
        Purpose: A StatefulSet is used to manage stateful applications that require stable identities, persistent storage, and ordered deployment and scaling.
        Pod Identity: Pods in a StatefulSet have a stable and unique identity. Each pod is given a unique name that persists across restarts (e.g., myapp-0, myapp-1, myapp-2).
        Use Case: StatefulSets are used for stateful applications like databases or applications that need to maintain persistent data (e.g., MongoDB, Cassandra).
        Pod Naming: Pods in a StatefulSet are named with an ordinal index (<name>-<ordinal>, e.g., myapp-0, myapp-1, myapp-2), which ensures that each pod has a unique and persistent identity.
        Storage: StatefulSets manage persistent volumes by associating each pod with a persistent volume that survives pod restarts.
        Scaling: Pods in a StatefulSet are created and scaled in a specific order. When scaling up, the new pods are added sequentially, and when scaling down, they are deleted in reverse order.
        Network Identity: StatefulSets provide each pod with a stable DNS hostname (e.g., myapp-0.myapp.default.svc.cluster.local), which is useful for stateful services like databases that need consistent network identities.
        Example: Databases (e.g., MySQL, Postgres) or any other service that requires persistent data storage.
        Example of a StatefulSet YAML:
        yaml
        Copy code
        apiVersion: apps/v1
        kind: StatefulSet
        metadata:
          name: my-statefulset
        spec:
          serviceName: "myapp"
          replicas: 3
          selector:
            matchLabels:
              app: myapp
          template:
            metadata:
              labels:
                app: myapp
            spec:
              containers:
              - name: myapp-container
                image: myapp-image
          volumeClaimTemplates:
          - metadata:
              name: myapp-storage
            spec:
              accessModes: [ "ReadWriteOnce" ]
              resources:
                requests:
                  storage: 1Gi
        Key Differences:
        Feature	ReplicaSet	StatefulSet
        Purpose	Manages stateless applications	Manages stateful applications with stable identities
        Pod Identity	Pods are interchangeable (no stable identity)	Pods have stable and unique identities (e.g., myapp-0)
        Pod Naming	Random names (e.g., myapp-abc123)	Ordinal names (e.g., myapp-0, myapp-1)
        Storage	No persistent storage, typically stateless	Persistent storage (each pod gets its own volume)
        Scaling	Pods are scaled independently	Pods are scaled in a specific order (sequential)
        Network Identity	No stable network identity	Stable DNS names for each pod
        Use Case	Stateless services (e.g., web servers, APIs)	Stateful services (e.g., databases, distributed apps)
        When to Use Which?
        ReplicaSet: When you are managing stateless applications that don’t require persistent storage or stable network identities, such as web servers or stateless services.
        StatefulSet: When your application requires a stable identity, ordered deployment, and persistent storage (e.g., databases or clustered services).
        In summary, ReplicaSet is more suitable for stateless workloads, whereas StatefulSet is designed for stateful applications that require persistent storage and stable identities.
        
        
## Does my ip would change in statefullset, if pod restarts

        In a StatefulSet, if a pod restarts, its IP address does not change by default.
        
        Here's how it works in more detail:
        
        IP Address Behavior in StatefulSet:
        Pod Identity: In a StatefulSet, each pod is given a stable and unique name (e.g., myapp-0, myapp-1). Each pod has a corresponding DNS name, which allows it to be consistently identified even if the pod restarts.
        Persistent IP: When a pod in a StatefulSet restarts, it is assigned the same IP address as it had before the restart. This is because StatefulSet uses a stable network identity for its pods.
        DNS Resolution:
        In addition to the stable IP, StatefulSet also provides stable DNS names for the pods (e.g., myapp-0.myapp.default.svc.cluster.local). These DNS names are resolved to the pod’s IP address, which remains consistent across pod restarts.
        Persistent Volumes:
        If your StatefulSet uses Persistent Volumes (PVs), these volumes are also maintained across pod restarts, ensuring that not only the IP but also the storage and state are retained.
        Summary:
        Yes, the pod will keep the same IP address even after it restarts in a StatefulSet. This is one of the key features of StatefulSets, which is useful for stateful applications like databases that rely on persistent network identities.

## What is the difference b/w statefullset and deamonset

      StatefulSet and DaemonSet are both Kubernetes controllers used to manage pods, but they serve different purposes and have key differences in how they manage and deploy pods. Here's a breakdown of the differences:
      
      1. Purpose
      StatefulSet: Manages stateful applications that require stable identities (names and IPs), persistent storage, and ordered deployment/termination. It is used for applications that maintain state, such as databases (e.g., MySQL, PostgreSQL).
      DaemonSet: Ensures that a copy of a pod is running on each node in the cluster, or a subset of nodes. It is used for tasks that require a pod to be present on all nodes, such as logging agents, monitoring agents, or network proxies.
      2. Pod Identity
      StatefulSet: Pods are given a stable, unique identity with persistent names and network IDs (e.g., myapp-0, myapp-1). The identity is preserved across restarts.
      DaemonSet: Pods do not have stable, unique identities like in StatefulSet. Each pod in the DaemonSet is typically independent, and its identity is tied to the node it is running on (e.g., node-name-pod).
      3. Pod Naming
      StatefulSet: Pods are given ordinal names with a stable suffix (e.g., myapp-0, myapp-1, myapp-2). This ensures each pod has a stable identity.
      DaemonSet: Pods are given names based on the node they are running on, but there is no ordinal numbering. Each pod runs on its respective node.
      4. Pod Scheduling
      StatefulSet: Pods are scheduled in a specific order. If a new pod is created, it is created sequentially (e.g., myapp-0 first, then myapp-1, etc.). This ensures ordered deployment and scaling.
      DaemonSet: Pods are scheduled to run on all nodes (or a subset of nodes based on labels), but there is no specific order in which the pods are scheduled.
      5. Scaling
      StatefulSet: Scaling a StatefulSet involves adding or removing pods in sequential order, ensuring that each pod gets a stable identity. When scaling down, pods are terminated in reverse order.
      DaemonSet: Scaling is handled automatically. If a new node is added to the cluster, a pod from the DaemonSet is automatically scheduled on that node. If a node is removed, the pod on that node is deleted.
      6. Storage
      StatefulSet: Can be used with Persistent Volumes (PVs), where each pod in the StatefulSet can have its own persistent storage that survives pod restarts. Each pod gets its own dedicated volume.
      DaemonSet: Pods do not typically use persistent storage unless configured otherwise. Each pod is independent, and their storage is not tied to any persistent volume.
      7. Use Cases
      StatefulSet: Used for applications that need stable identities, persistent storage, and ordered deployment, such as databases (e.g., MongoDB, MySQL, Cassandra) or distributed systems that require stable network identities.
      DaemonSet: Used for cluster-wide services that need to run on all or a specific set of nodes, such as logging, monitoring, networking agents, or security tools that need to be deployed to every node in the cluster.
      8. Network Identity
      StatefulSet: Provides stable DNS names for each pod (e.g., myapp-0.myapp.svc.cluster.local) that can be used by other pods for consistent networking and communication.
      DaemonSet: Does not provide stable DNS names for each pod. Pods are usually identified by the node they are on.
      9. Termination Behavior
      StatefulSet: Pods are terminated in reverse order (i.e., last pod created is terminated first) to maintain order.
      DaemonSet: Pods are terminated as needed, and there is no specific termination order.
      Summary of Key Differences:
      Feature	StatefulSet	DaemonSet
      Purpose	Manages stateful applications with stable identities	Ensures a pod runs on every node in the cluster
      Pod Identity	Stable, unique identity (e.g., myapp-0)	No stable identity, tied to the node
      Pod Naming	Ordinal naming (e.g., myapp-0, myapp-1)	Node-based naming, no ordinal numbering
      Pod Scheduling	Sequential, ordered pod deployment	Pods scheduled on every node
      Scaling	Sequential scaling, adding/removing pods in order	Automatic scaling across nodes
      Storage	Persistent Volumes with unique storage for each pod	No persistent storage, unless configured
      Use Case	Stateful applications (e.g., databases, clustered services)	Cluster-wide services (e.g., logging, monitoring)
      Network Identity	Stable DNS names for each pod	No stable DNS names
      Termination Behavior	Ordered pod termination (reverse order)	No specific order
      When to Use Each:
      StatefulSet: Use for applications that require stable identities, persistent storage, and ordered deployment (e.g., databases, distributed systems).
      DaemonSet: Use for applications that need to run on every node or a subset of nodes, such as monitoring tools, logging agents, or network proxies.
      In summary, StatefulSet is suitable for applications requiring stable identities and persistent storage, whereas DaemonSet is used for running a pod on every node, typically for monitoring, logging, or other cluster-wide services.
              
        
## AWS RDS(Creating an end-to-end infrastructure on AWS that includes setting up RDS (Relational Database Service) with best practices for security, data integration, and optimal usage for various environments involves several steps.)

      Creating an end-to-end infrastructure on AWS that includes setting up RDS (Relational Database Service) with best practices for security, data integration, and optimal usage for various environments involves several steps. Here’s a practical example of how to approach this:
      
      Step 1: Set Up AWS Infrastructure
      Create a VPC (Virtual Private Cloud):
      
      A VPC provides an isolated network to launch AWS resources.
      Subnets: Create at least two subnets in different Availability Zones (AZs) for high availability.
      Internet Gateway (IGW): Attach to your VPC if you need to access the internet.
      Route Tables: Set up route tables for routing traffic to the internet (for public subnets).
      NAT Gateway: If you have private subnets, use a NAT Gateway for outbound internet access.
      Create Security Groups:
      
      Security groups act as firewalls for your AWS resources, controlling inbound and outbound traffic.
      Example for RDS: Allow access from specific IPs or EC2 instances, but restrict broad access.
      Example for EC2: Allow SSH (port 22) from your IP and HTTP/HTTPS (ports 80/443) for web traffic.
      Create IAM Roles and Policies:
      
      Create an IAM role for EC2 instances to allow interaction with RDS and other AWS services.
      Attach an IAM policy to your role to restrict access to only necessary resources (e.g., RDS access, S3 access).
      Step 2: Deploy AWS RDS
      Choosing the Right RDS Instance Type:
      
      General Purpose (e.g., db.t3.medium): Suitable for small to medium workloads, low-cost option.
      Memory Optimized (e.g., db.r5.large): Use for applications that require more memory (e.g., high-traffic websites).
      Burstable Performance (e.g., db.t3.micro): Cost-effective and good for development or light workloads.
      Provisioned IOPS (e.g., db.m5.large with io1): For high-performance, low-latency database needs (e.g., high-volume transaction applications).
      Multi-AZ: For high availability and disaster recovery. This creates a standby replica of the DB in another AZ.
      Deploy an RDS Instance (example for MySQL):
      
      Engine: MySQL or PostgreSQL, depending on your application requirements.
      Instance Size: Select based on the expected load (e.g., db.m5.large).
      Storage Type: Use General Purpose SSD (gp2) for standard use, or Provisioned IOPS (io1) for high performance.
      Encryption: Enable encryption for data at rest, ensuring compliance with security requirements.
      Backup: Enable automated backups with a retention period that suits your environment (e.g., 7 days).
      Monitoring: Enable enhanced monitoring and CloudWatch alarms for tracking RDS performance metrics like CPU usage, read/write latency, etc.
      Network Configuration for RDS:
      
      Subnets: Ensure your RDS instance is placed in private subnets for security.
      VPC Security Group: Attach the RDS security group, which only allows inbound connections from your application servers or specific IP addresses.
      Step 3: Secure the RDS Database
      Encryption:
      
      Use KMS (Key Management Service) for encrypting data at rest in RDS.
      Enable SSL/TLS for encrypted communication between your application and RDS instance.
      Access Control:
      
      Use IAM authentication for connecting to RDS instances to avoid using traditional database passwords.
      Security Groups: Only allow access to the RDS instance from your EC2 instances or other trusted IP addresses.
      Enable RDS IAM Database Authentication to use IAM roles for accessing RDS instead of hardcoded credentials.
      Backup and Snapshot:
      
      Regular automated backups ensure that you can restore the database to a previous state.
      Manual Snapshots: Take manual snapshots before performing major changes, like schema modifications or upgrades.
      Enable Point-in-Time Recovery (PITR) to restore the database to any specific time within the retention period.
      VPC Peering / PrivateLink:
      
      If you need to connect your RDS instance to other VPCs (e.g., for data integration), you can use VPC Peering or AWS PrivateLink.
      Step 4: Data Integration and Migration
      Data Migration:
      
      If you're migrating data to AWS RDS, you can use AWS Database Migration Service (DMS) to migrate data from on-premises or another cloud environment to RDS.
      Use RDS Read Replicas to replicate data across multiple regions for disaster recovery or cross-region data synchronization.
      Data Integration:
      
      For ETL (Extract, Transform, Load) workflows, consider using AWS Glue or Amazon RDS Data API for connecting RDS with other AWS services like S3, Redshift, etc.
      For cross-environment data integration (e.g., EC2 to RDS), use AWS Direct Connect for high-bandwidth, low-latency connections.
      Step 5: Monitoring and Maintenance
      Performance Monitoring:
      
      Use Amazon CloudWatch to track RDS performance metrics, such as CPU, memory, disk I/O, and network throughput.
      Set up CloudWatch Alarms to notify you when metrics exceed predefined thresholds, allowing you to take action before performance degrades.
      RDS Events:
      
      Enable RDS Event Subscriptions to receive notifications about important events (e.g., failover, backup completion, storage limit reached).
      Use these alerts to trigger AWS Lambda functions to automate corrective actions (e.g., scaling the instance).
      Patch Management:
      
      Enable RDS Maintenance Window to schedule database updates and patches during off-peak hours to reduce impact on application availability.
      Step 6: Best Practices for Environment Suitability
      Development/Testing Environment:
      
      Use smaller instance types and lower-cost storage options (e.g., db.t3.micro and gp2 storage) to save costs.
      Set up a staging environment identical to production to test changes before applying them.
      Production Environment:
      
      Multi-AZ deployments for high availability and automatic failover.
      Use Provisioned IOPS for high-performance applications with heavy database transactions.
      Set up automatic backups with a retention period of at least 7 days, and use point-in-time recovery.
      Security:
      
      Ensure that IAM roles and policies are tightly scoped to limit the access each resource has to other services.
      Apply the least privilege principle by limiting access to the RDS database (e.g., only allow EC2 instances with a certain role to connect).
      Cost Optimization:
      
      Use Reserved Instances (RDS) if your workload is predictable to reduce costs.
      Enable storage auto-scaling to automatically adjust storage based on usage and avoid over-provisioning.
      Step 7: Example Terraform Code for RDS Deployment
      Here is an example Terraform script to create an RDS instance:
      
      hcl
      Copy code
      resource "aws_db_instance" "example" {
        allocated_storage = 20
        storage_type      = "gp2"
        engine            = "mysql"
        engine_version    = "8.0.23"
        instance_class    = "db.t3.medium"
        name              = "mydatabase"
        username          = "admin"
        password          = "yourpassword"
        db_subnet_group_name = aws_db_subnet_group.example.name
        multi_az          = true
        publicly_accessible = false
        backup_retention_period = 7
        final_snapshot_identifier = "mydb-final-snapshot"
      
        vpc_security_group_ids = [aws_security_group.db_sg.id]
      
        tags = {
          Name = "My RDS Instance"
        }
      }
      
      resource "aws_db_subnet_group" "example" {
        name       = "mydb-subnet-group"
        subnet_ids = [aws_subnet.subnet1.id, aws_subnet.subnet2.id]
      
        tags = {
          Name = "My DB Subnet Group"
        }
      }
      Conclusion
      By following this approach, you can design and implement a secure, scalable, and highly available RDS infrastructure on AWS. This involves selecting the right instance types, securing your RDS instances, integrating with other AWS services, and maintaining the system with proper monitoring and backups. Each decision depends on the environment (development, production) and specific application requirements.        
              
What is QOS(Quality of Service)
------------------------------- 

    In Kubernetes, QoS (Quality of Service) refers to the classification of pods based on the resource requests and limits they define. It helps the Kubernetes scheduler and kubelet determine how to prioritize and allocate resources to pods, especially during times of resource contention.
    
    QoS classes ensure that critical workloads are given priority over less critical ones and manage how pods are treated when resources are limited.
    
    Kubernetes QoS Classes:
    There are three QoS classes:
    
    1. Guaranteed
    Criteria:
    The pod's resource requests and limits are equal for all containers.
    Both cpu and memory must have defined requests and limits.
    Priority:
    Highest priority.
    These pods are the least likely to be evicted when a node is under resource pressure.
    Example:
    
    yaml
    Copy code
    containers:
    - name: app
      resources:
        requests:
          memory: "1Gi"
          cpu: "500m"
        limits:
          memory: "1Gi"
          cpu: "500m"
    2. Burstable
    Criteria:
    The pod has resource requests specified, but limits and requests are not equal.
    One or more containers may have only requests or limits partially specified.
    Priority:
    Medium priority.
    These pods can use additional resources if available but may be evicted if the node runs out of resources.
    Example:
    
    yaml
    Copy code
    containers:
    - name: app
      resources:
        requests:
          memory: "512Mi"
          cpu: "250m"
        limits:
          memory: "1Gi"
          cpu: "1"
    3. BestEffort
    Criteria:
    The pod does not define any resource requests or limits for its containers.
    Priority:
    Lowest priority.
    These pods are most likely to be evicted during resource contention.
    They can only use resources when no other higher-priority pods need them.
    Example:
    
    yaml
    Copy code
    containers:
    - name: app
    Why QoS is Important:
    Resource Management: Helps Kubernetes allocate resources efficiently across workloads.
    Eviction Handling: Determines which pods are evicted first during resource shortages.
    Cluster Stability: Ensures critical applications remain functional under high load.
    In short, QoS helps Kubernetes balance workloads and ensure high-priority applications are not disrupted by lower-priority ones.
    


## What is the difference b/w side car and init container

The difference between Sidecar and Init Container in Kubernetes lies in their purpose, lifecycle, and how they interact with the main application container. Here’s a detailed breakdown:

Init Container 
--------------

1. Purpose
   Preparation Tasks: Used to run initialization tasks before the main application container starts.

   Examples: Setting up configurations, downloading dependencies, or checking external services.

2. Lifecycle
   Runs sequentially before the main container starts. The main container doesn’t start until all Init Containers complete successfully.

   Stops after completing its task.

3. Interaction with the Main Container
   Has no direct interaction with the main container after completing its task.

Sidecar
-------

1. Purpose
   Enhancement/Support: Runs alongside the main application container to provide additional functionality or support.

   Examples: Logging, monitoring, proxying traffic, or data synchronization.

2. Lifecycle
   Runs concurrently with the main container as part of the pod and remains active throughout the pod's lifecycle.

   Runs continuously and only stops when the pod is terminated.

3. Interaction with the Main Container
   Works in tandem with the main container to enhance its functionality.

    
    4. Use Cases
    Init Container:
    Perform one-time setup tasks:
    Configure files or secrets.
    Initialize database schemas or populate data.
    Wait for an external service (e.g., database) to become available.
    Sidecar:
    Provide ongoing support:
    Proxy traffic (e.g., Envoy or Istio sidecar).
    Log aggregation (e.g., Fluentd or Logstash sidecar).
    Run monitoring agents.
    Synchronize data in real-time (e.g., syncing files from a shared volume).
    5. Configuration Examples
    Init Container Example:
    yaml
    Copy code
    apiVersion: v1
    kind: Pod
    metadata:
      name: init-container-example
    spec:
      initContainers:
      - name: init-container
        image: busybox
        command: ["sh", "-c", "echo Initializing... && sleep 10"]
      containers:
      - name: main-app
        image: nginx
    Key Points:
    
    The Init Container runs first, waits 10 seconds, and completes before nginx starts.
    Sidecar Example:
    yaml
    Copy code
    apiVersion: v1
    kind: Pod
    metadata:
      name: sidecar-example
    spec:
      containers:
      - name: main-app
        image: nginx
      - name: log-aggregator
        image: fluentd
        volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
      volumes:
      - name: logs
        emptyDir: {}
    Key Points:
    
    The Fluentd sidecar runs alongside nginx, collecting logs from a shared volume.
    Summary
    Aspect	Init Container	Sidecar
    Runs before main	Yes	No
    Runs concurrently	No	Yes
    One-time setup	Yes	No
    Continuous support	No	Yes
    In essence, Init Containers prepare the environment, while Sidecars extend and support the main application.


## Facing issue that cloudrun is not running the container

  Remember, always configure the container to listen on the same port that the application expects traffic. I faced an issue where my application was running on port 8000, but I mistakenly opened port 8080. As a result, Cloud Run was sending traffic to port 8080, receiving no response, and failing repeatedly. To avoid this, always ensure the container's listening port matches the application's expected port
     
## What is the difference b/w terraform reconfigure and terraform migration

      Terraform reconfigure
      
      Uses the terraform init -reconfigure command to reconfigure a backend while ignoring any saved configuration. This option is used when the backend configuration file is corrupt or when you want to start fresh with a new backend.
      
      Terraform migration
      
      Uses the terraform init -migrate-state command to reconfigure the backend and attempt to migrate any existing state. 
      When you change your backend configuration, Terraform will automatically detect the changes and ask if you want to migrate your existing state to the new configuration. You can respond "no" if you're just reconfiguring the same backend. 
      The goal of Terraform migration is to increase the velocity and reliability of deploying and managing modern cloud architectures.


## How to create AWS account 

    linke: aws.amazon.com
    
    steps:
    -----
    
    create an aws account>root user email address(give email), AWS account name(give account name)>verify email address(now check email and verify email address. and get verfication code from email use this in next page, it will verify your email)>select personal use , give your address information>here you need to give your atm card information, it will basically verifying that card account no is correct or not.> next you need to give you phone na and enter correct capacha,(aws will send verfication code on your mobile.) you that code here>next select aws basic plan, now it will open the aws console..

AWS CodeStar
-----------
  
  AWS CodeStar is a cloud service designed to make it easier to develop, build, and deploy applications on AWS by simplifying the setup of your entire development project. AWS CodeStar includes project templates for common development platforms to enable provisioning of projects and resources for coding, building, testing, deploying, and running your software project.
  
  The key benefits of the AWS CodeStar service are:
  
  - Easily create new projects using templates for Amazon EC2, AWS Elastic Beanstalk, or AWS Lambda using five different programming languages; JavaScript, Java, Python, Ruby, and PHP. By selecting a template, the service will provision the underlying AWS services needed for your project and application.
  
  - Unified experience for access and security policies management for your entire software team. Projects are automatically configured with appropriate IAM access policies to ensure a secure application environment.
  
  - Pre-configured project management dashboard for tracking various activities, such as code commits, build results, deployment activity and more.
  
  - Running sample code to help you get up and running quickly enabling you to use your favorite IDEs, like Visual Studio, Eclipse, or any code editor that supports Git.
  
  - Automated configuration of a continuous delivery pipeline for each project using AWS CodeCommit, AWS CodeBuild, AWS CodePipeline, and AWS CodeDeploy.
  
  - Integration with Atlassian JIRA Software for issue management and tracking directly from the AWS CodeStar console
  
  With AWS CodeStar, development teams can build an agile software development workflow that not only increases the speed in which teams can deploy software and bug fixes, but also enables developers to build software that is more inline with customers’ requests and needs
    
## Identity base policy can attach with (users,groups and role it cannot directly attach with aws service) but resource base policy can directly attach with AWS services.
    
    Yes, your understanding is correct! Here's a clearer breakdown to confirm and simplify your explanation:
    
    Identity-Based Policies:

    What are they?
    Policies that are attached to IAM users, groups, or roles to grant them permissions.
    
    Can they attach directly to AWS services?
    No, identity-based policies cannot be directly attached to an AWS service.
    
    Instead, you attach them to an IAM role, and then that role can be assigned to the AWS service (e.g., an EC2 instance or Lambda function).
    

    Resource-Based Policies:
    
    What are they?
    Policies that are directly attached to an AWS resource (like an S3 bucket, SNS topic, or SQS queue).
    
    Can they attach directly to AWS services?
    Yes, resource-based policies are directly attached to AWS resources. For example:
    S3 bucket policies
    KMS key policies
    SNS topic policies
    
    Key Difference:
    Identity-Based Policies:
    Attach to IAM identities (users, groups, or roles).
    Require a role to mediate between the AWS service and the identity.
    
    Resource-Based Policies:
    Attach directly to an AWS resource.
    No need for a separate role.
    
    Your Statement:
    "Identity-based IAM policies can be attached to users, groups, or IAM roles but cannot be directly attached to an AWS service."
    "Resource-based IAM policies can be directly attached to AWS services."


## REMEMBER: There are multiple ways of assigning the variables to different dev Ops tools\

like:
----

- If you are running terraform locally with terraform init and plan, apply command, **then you can pass variables through variable.tf or you can define variable on  variable.tfvar file and .tfvar file passing that variable to variable.tf but during executing terraform plan and terraform apply you need to select .tfvar file for variable. so terraform will get to know that variable are define in .tfvar file and terraform will pass these variable to main.tf** or you can also pass variable by running **terraform plan -vars=name=values flags and terraform apply -vars=name=values  flag** it will take the most precidence. 

- if you are defining terraform plan -vars= and terraform apply -vars= command in bash file for making it automate, and want to get values outside the bash file.. when you can use "$" sign with value.

  like this:

  !#/bin/bash
  terraform plan -vars="project-id=$project-id" 
  terraform apply -vars="project-id=$project-id"

  You also need to export enviroment variable on terminal using "export" command. where you going to run bash file.

  like this:
  ---------

  export project-id=values of project id   

  once terraform run plan and apply command , it will get the variable values from terminal what you have give during using export command..

- if you are trying to run this with docker, then you can use **-e** flag with docker run command for passing environment variable to docker container

   like this:
   ---------
    docker run \
      -e PROJECT_ID="world-learning-400909" \
      -e PROJECT_NUMBER="22927148231" \
      -e DEFAULT_OWNER_EMAIL="muhammadhassanb122@gmail.com" \
      -e REGION="us-central1" \
      -e LOCATION="us-central1-c" \
      -e VPC_NAME="uscentral-vpc01-100008" \
      -e SUBNET_NAME="uscentral-disearch-vpc01-subnet1000024" \
      -e GKE_SUBNET_NAME="uscentral-disearch-vpc01-subnet1010016-gke" \
      -e DB_INSTANCE_DISK_SIZE="20" \
      -e DB_USERNAME="postgres" \
      -e DB_PASSWORD="CKhEJZH[uFS;%=Mg8iwueVm&x" \
      -e CLOUD_RUN_SERVICE_NAME="my-cloud-run-service" \
      -e GKE_CLUSTER_NAME="disearch-cluster" \
      -e GKE_NODES_MACHINE_TYPE="e2-highmem-4" \
      -e STORAGE_BUCKET_NAME="disearch-storage-bucket" \
      -e STRIPE_SECRET_KEY="" \
      -e ARETEC_ADMIN="secret123" \
      -e LOGS_URL="http://ipaddress:24224/myapp.logs" \
      -e OPENAI_KEY="" \
      -e OPENAI_API_KEY="" \
      -e SERVICE_ACCOUNT_KEY='{service account }' \
     -itd gcr.io/aretecinc-public/disearch/deployer/terraform-deployer:latest
    

- You can pass enviroment variable to kubernetes pod through "configmap" and "secret" as env.  and you application  or bash will get these variables from env.   
  
  
## Remember if you are facing issues on deploying any resource on kubernetes or ecs..

- then try to run this locally or in docker container. and see the issue by seeing docker logs  or agr issue trace na hotu, docker container k sleep time increase ker do, 

    - Dockerfile ma  **cmd "/bin/bash" "10000"** , or container ma jkr file ko dekhi sahi i hn, jo ap na local terminal sa build ker k send ki thi, some time files hi sahi ni i hoti.. 
    - Sometime permission k issue hota ha. jo k docker container ma jkr **ls -lrt** kr k file permission k issue pta chalta ha..
    - you can also use **ls** in docker file for seeing that docker is including files or not.. during building images...
    - or agr ap dekhna chahty hn k terraform apky cloud k sath container ma sa connect ha k ni, because container k under sa terraform na cloud ma resource ko provision krna ha..  then ap bucket list ki command use ker k verify ker sakhty hn **gsutils**. is sa apko verify hojye ga k container cloud k sath connect ha, so agr connect ha tu wo baki resources b provision ker lye ga..  
    
  
## PGBOUNCER MODES

      PgBouncer, a lightweight PostgreSQL connection pooler, operates in three primary pooling modes that define how client connections interact with database server connections. Here’s an overview of each mode:
      
      1. Session Pooling
      Description: Each client connection gets a dedicated server connection for the duration of the session.
      Behavior:
      When a client connects, PgBouncer assigns a database connection that remains reserved until the client disconnects.
      Once the client disconnects, the database connection returns to the pool.
      Use Case: Best suited for applications with long-running sessions or that rely heavily on session-level features like prepared statements or temporary tables.
      
      3. Transaction Pooling
      Description: Each client connection gets a server connection only for the duration of a transaction.
      Behavior:
      After a transaction completes, the database connection is returned to the pool immediately.
      Between transactions, the client connection is idle, but it doesn't hold a server connection.
      Limitations:
      Features like session-based prepared statements, advisory locks, and temporary tables are not supported since the server connection is reset after each transaction.
      Use Case: Best for applications with short-lived transactions and stateless interactions with the database.
      
      5. Statement Pooling
      Description: Each client connection gets a server connection only for the execution of a single SQL statement.
      Behavior:
      After executing the statement, the connection is returned to the pool immediately.
      Limitations:
      Features requiring session or transaction contexts (e.g., multi-statement transactions, cursors, prepared statements) are not supported.
      Use Case: Ideal for applications where every query is independent, such as read-heavy analytics workloads or specific APIs.
      Choosing the Right Mode:
      The mode you choose depends on your application's requirements:
      
      Session Pooling: For applications requiring persistent session state.
      Transaction Pooling: For most modern web applications that can handle stateless behavior.
      Statement Pooling: For use cases with completely independent single statements.
      You can configure the pooling mode in the PgBouncer configuration file (pgbouncer.ini) by setting:
      
      
      pool_mode = <session|transaction|statement>
      Replace <session|transaction|statement> with your desired pooling mode.
      

## Bash scripting provides several types of loops to handle repetitive tasks. Here’s a list of the main loops available in bash scripts:

    1. for Loop
    Iterates over a list of items or a range of numbers.
    
    # Loop through a list
    for item in apple banana cherry; do
      echo "Fruit: $item"
    done

    # Loop through a range
    for i in {1..5}; do
      echo "Number: $i"
    done

    2. while Loop
    Repeats as long as a condition is true.
    
    count=1
    while [ $count -le 5 ]; do
      echo "Count: $count"
      count=$((count + 1))
    done

    3. until Loop
    Repeats until a condition becomes true (opposite of while).
    
    count=1
    until [ $count -gt 5 ]; do
      echo "Count: $count"
      count=$((count + 1))
    done

    4. select Loop
    Used for creating menus in a script, allowing a user to choose an option.
    
    PS3="Choose a fruit: "
    select fruit in apple banana cherry; do
      echo "You selected: $fruit"
      break
    done
    
    5. Infinite Loop
    Created using while true or for (( ; ; )) to run indefinitely until explicitly stopped.
    
    # Infinite while loop
    while true; do
      echo "Running..."
      sleep 1
    done

    # Infinite for loop
    for (( ; ; )); do
      echo "This goes on forever!"
      sleep 1
    done
    Summary of Loops:
    Loop Type	Purpose
    for	Iterate over a list or range of values.
    while	Execute as long as a condition is true.
    until	Execute until a condition becomes true.
    select	Build interactive menus.
    Infinite	Run indefinitely (requires manual break).
    Each loop serves a specific purpose, and you can choose the one that best fits your use case! Would you like examples for a specific scenario?

## if vs case statement

    In bash scripting, if and case statements are both used for conditional execution, but they serve different purposes and are used in different situations.
    
    1. if Statement
    Purpose: Used to evaluate boolean conditions. It executes commands based on whether a condition is true or false.
    Best For: Simple conditions or when conditions involve comparisons (==, >, <, etc.).
    Syntax:
 
    if [ condition ]; then
      # Commands to execute if true
    elif [ another_condition ]; then
      # Commands to execute if another condition is true
    else
      # Commands to execute if all conditions are false
    fi
    Example:
    bash
    Copy code
    number=5
    if [ $number -gt 3 ]; then
      echo "Number is greater than 3"
    elif [ $number -eq 3 ]; then
      echo "Number is equal to 3"
    else
      echo "Number is less than 3"
    fi
   

   2. case Statement
    Purpose: Used to match a single value against multiple patterns. It executes a block of code based on the first matching pattern.
    Best For: Handling multiple specific values or patterns (e.g., menu options or file extensions).
    Syntax:

    case $variable in
      pattern1)
        # Commands to execute for pattern1
        ;;
      pattern2)
        # Commands to execute for pattern2
        ;;
      *)
        # Commands to execute if no patterns match
        ;;
    esac
   
   Example:
  
    fruit="apple"
    case $fruit in
      apple)
        echo "This is an apple"
        ;;
      banana)
        echo "This is a banana"
        ;;
      orange)
        echo "This is an orange"
        ;;
      *)
        echo "Unknown fruit"
        ;;
    esac

    Key Differences:
    Feature	if Statement	case Statement
    Condition Type	Evaluates conditions (e.g., comparisons, tests).	Matches values or patterns.
    Best Use Case	Comparing numbers, strings, or logical conditions.	Handling multiple specific values or patterns.
    Syntax Simplicity	Can get complex with multiple conditions.	Simpler for matching multiple values.
    Wildcard Support	Does not natively support wildcards.	Supports wildcards (e.g., *, ?).
    
    Real-Life Analogy:
    if Statement: "If it's raining, take an umbrella. If it's cloudy, carry a jacket. Otherwise, go without extra gear." (Conditions-based decisions).
    case Statement: "What day is it? If it's Monday, go to work. If it's Saturday, relax. If it's Sunday, plan for the week." (Value-based decisions).
    When to Use:
    
    Use if when you need logical comparisons or boolean conditions.
    Use case when you need to handle specific values or patterns.
    Let me know if you'd like more examples! 😊

## module for string and integre
      
      In Ansible, there isn’t a specific module solely dedicated to handling strings or integers. However, you can perform operations on strings and integers using Jinja2 templating and a combination of modules and filters. Below are examples and explanations of how you can work with strings and integers in Ansible.
      
      1. String Operations
      Use Case: Concatenation, replacing parts of a string, or formatting text.
      Relevant Filters: lower, upper, replace, length, split, etc.
      Example: String Manipulation
  
      - name: String manipulation example
        hosts: localhost
        tasks:
          - name: Convert string to uppercase
            debug:
              msg: "{{ 'hello world' | upper }}"
              
          - name: Replace part of a string
            debug:
              msg: "{{ 'hello world' | replace('world', 'Ansible') }}"
      
          - name: Find string length
            debug:
              msg: "Length of string: {{ 'hello world' | length }}"
     
     2. Integer Operations
      Use Case: Arithmetic operations, comparisons, or formatting numbers.
      Relevant Filters: abs, round, int, to_json, to_yaml, etc.
      Example: Integer Manipulation
  
      - name: Integer manipulation example
        hosts: localhost
        tasks:
          - name: Add two integers
            debug:
              msg: "Sum: {{ 5 + 3 }}"
              
          - name: Convert string to integer and perform math
            debug:
              msg: "Result: {{ '10' | int * 2 }}"
              
          - name: Round a floating-point number
            debug:
              msg: "Rounded value: {{ 5.678 | round(1) }}"
      3. Combining String and Integer Operations
      You can combine string and integer operations in tasks. For example, you might format a string with dynamic integer values.
      
      Example: Dynamic Formatting

      - name: Combine string and integer operations
        hosts: localhost
        tasks:
          - name: Generate a dynamic message
            vars:
              name: "Hassan"
              age: 30
            debug:
              msg: "Hello {{ name }}, you are {{ age }} years old!"
      4. Modules for String and Integer Handling
      While these modules don’t directly work on strings/integers, they are helpful for managing related tasks:
      
      set_fact
      Purpose: Define variables dynamically, including strings and integers.
      Example:

      - name: Set a dynamic variable
        hosts: localhost
        tasks:
          - name: Define a variable
            set_fact:
              message: "Hello {{ 'World' | upper }}"
          - name: Use the variable
            debug:
              msg: "{{ message }}"
      template
      Purpose: Render strings and integers in configuration files using Jinja2.
      Example:

      - name: Render a template with variables
        hosts: localhost
        tasks:
          - name: Generate a file
            template:
              src: example_template.j2
              dest: /tmp/example_output.txt
      Content of example_template.j2:
      
      jinja2
      Copy code
      Name: {{ name }}
      Age: {{ age }}
      5. Real-Life Use Cases
      String: Formatting a hostname (host123.domain.com) dynamically based on variables.
      Integer: Calculating the total size of disk space or creating loops based on numbers.
      Let me know if you'd like help with a specific example! 😊

## How to clone code commit repo on local terminal
    
    To clone a repository from AWS CodeCommit, follow these steps:
    
    1. Prerequisites
    Before cloning the repository, ensure the following:
    
    AWS CLI is Installed and Configured:
    
    Install AWS CLI:
    
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install
    
    Configure AWS CLI with your credentials:
    
    aws configure
    Enter:
    Access Key ID
    Secret Access Key
    Default region (e.g., us-east-1)
    Default output format (e.g., json)
    
    Git is Installed:
    
    Install Git:
    
    sudo apt install git -y  # For Debian-based systems
    sudo yum install git -y  # For RHEL-based systems
    
    IAM Permissions:
    
    Ensure your IAM user/role has the AWSCodeCommitReadOnly or AWSCodeCommitFullAccess policy.
    
    SSH Key or HTTPS Access:
    
    For SSH Access:
    generate and Add your SSH public key to AWS CodeCommit via the IAM Console under Security Credentials.
    
    command to generate public ssh key: ssh-keygen -t rsa -b 4096 -C "your_email@example.com" 

    now copied public key and add your ssh public key to aws codecommit via the IAM Console under Security Credentials.

    for this copied public key , then go to IAM user > security credentials > SSH public keys for AWS CodeCommit and upload ssh public key here,

    For HTTPS Access:
    Use AWS CLI-generated credentials or configure Git credentials.
    
    2. Clone the Repository
    Option 1: Using HTTPS
    Get the repository URL:
    
    Go to the AWS Management Console → CodeCommit → Open your repository → Copy the HTTPS URL.
    
    Run the clone command:
    
    
    git clone https://git-codecommit.<region>.amazonaws.com/v1/repos/<repository-name>
    Replace:
    
    <region>: The AWS region where the repository is hosted (e.g., us-east-1).
    <repository-name>: Your repository name.
    If prompted, enter your username and password:
    
    Username: AWS
    Password: Your IAM credentials or AWS CLI credentials.
                    
                
                
                
                
                
                
                
                
                                  
                              
                               
