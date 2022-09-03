---
title: Getting started with GKE
date: 2022-07-07T06:52:36+00:00
image_webp: "/images/cloud.jpg"
image: "/images/cloud.jpg"
author: Sandeep Reddy
description: Google cloud

---
Getting Started with GKE

## Objectives

1. Create a Google Kubernetes Engine cluster containing several containers, each containing a web server.
2. Place a load balancer in front of the cluster and view its contents.

***

### Task 1. Sign in to the Google Cloud

### Task 2. Confirm that needed APIs are enabled

1. Make a note of the name of your Google Cloud project.
2. In the Google Cloud Console, on the Navigation menu (Navigation menu icon), click APIs & Services.
3. Scroll down in the list of enabled APIs, and confirm that both of these APIs are enabled:

   Kubernetes Engine API

   Container Registry API

If either API is missing, click Enable APIs and Services at the top. Search for the above APIs by name and enable each for your current project. (You noted the name of your GCP project above.)

### Task 3. Start a Kubernetes Engine cluster

Click on Cloud Shell icon. Click Continue.

    export MY_ZONE=us-central1-a

#### Start a Kubernetes cluster managed by Kubernetes Engine. Name the cluster webfrontend and configure it to run 2 nodes:

    gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2

It takes several minutes to create a cluster as Kubernetes Engine provisions virtual machines for you.

After the cluster is created, check your installed version of Kubernetes using the kubectl version command:

    kubectl version

The gcloud container clusters create command automatically authenticated kubectl for you.

View your running nodes in the GCP Console. On the Navigation menu (Navigation menu icon), click Compute Engine > VM Instances.

Your Kubernetes cluster is now ready for use.

### Task 4. Run and deploy a container

From your Cloud Shell prompt, launch a single instance of the nginx container. (Nginx is a popular web server.)

    kubectl create deploy nginx --image=nginx:1.17.10
    
    kubectl get pods

#### Expose the nginx container to the Internet:

    kubectl expose deployment nginx --port 80 --type LoadBalancer

Kubernetes created a service and an external load balancer with a public IP address attached to it. The IP address remains the same for the life of the service. Any network traffic to that public IP address is routed to pods behind the service: in this case, the nginx pod.

#### View the new service:

    kubectl get services

You can use the displayed external IP address to test and contact the nginx container remotely.

It may take a few seconds before the External-IP field is populated for your service. This is normal. Just re-run the kubectl get services command every few seconds until the field is populated.

Open a new web browser tab and paste your cluster's external IP address into the address bar. The default home page of the Nginx browser is displayed.

#### Scale up the number of pods running on your service:

    kubectl scale deployment nginx --replicas 3

Scaling up a deployment is useful when you want to increase available resources for an application that is becoming more popular.

#### Confirm that Kubernetes has updated the number of pods:

    kubectl get pods

#### Confirm that your external IP address has not changed:

    kubectl get services