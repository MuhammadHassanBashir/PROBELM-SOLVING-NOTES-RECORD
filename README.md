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

**Docker** if code is running on 8000(for this see code file like main.py for python) then in dockerfile expose port for container should set on same 8000 port regardless whatever is set in entrypoint(set same code pod here.) and for starting the container you can map any external port to same internal container port during running the container using the docker run command...

  docker run --port externalport:internetport(containerport, same on which code is running)     
    

  



  
