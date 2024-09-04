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

  
  

  
