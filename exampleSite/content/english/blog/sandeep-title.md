+++
author = "sandeep"
date = 2022-08-27T18:30:00Z
description = "kubernetes"
image = "/images/k8s-1.jpg"
image_webp = "/images/k8s.jpg"
title = "Kubernetes 3 node setup"

+++
Kubernetes 3 node setup

## Docker setup:

3 nodes(servers) with Ubuntu 18.04 Bionic Beaver LTS.

One server being “Kube Master” and remaining 2 servers as “Kube Node 1” and “Kube Node 2”.

The first step in setting up a new cluster is to install a container runtime such as Docker.

We will be installing Docker on our three servers in preparation for standing up a Kubernetes cluster.

**Commands:**

    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -sudo add-apt-repository \”deb [arch=amd64] https://download.docker.com/linux/ubuntu \$(lsb_release -cs) \stable”
    
    sudo apt-get update
    
    sudo apt-get install -y docker-ce=18.06.1~ce~3–0~ubuntu
    
    sudo apt-mark hold docker-ce
    
    sudo docker version
    

## K8s setup

Now that Docker is installed, we are ready to install the Kubernetes components.

Install Kubeadm, Kubelet, and Kubectl on all three playground servers.

After this bootstrap the cluster. Here are the commands used to install the Kubernetes components. Run these on all three servers.

**NOTE:** There are some issues being reported when installing version 1.12.2–00 from the Kubernetes ubuntu repositories. You can work around this by using version 1.52.7–00 for kubelet, kubeadm, and kubectl.

**Commands:**

    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add —
    
    cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.listdeb https://apt.kubernetes.io/ kubernetes-xenial main EOF
    
    sudo apt-get update
    
    sudo apt-get install -y kubelet=1.15.7–00 kubeadm=1.15.7–00 kubectl=1.15.7–00
    
    sudo apt-mark hold kubelet kubeadm kubectl
    
    kubeadm version
    

## K8s cluster bootstrap

Now we are ready to get a real Kubernetes cluster up and running!

Now we will bootstrap the cluster on the Kubemaster node. Then, we will join each of the two worker nodes to the cluster, forming an actual multi-node Kubernetes cluster.

Here are the **commands** used in this lesson:

On the Kube master node, initialize the cluster:

    sudo kubeadm init — pod-network-cidr=10.244.0.0/16

That command may take a few minutes to complete.When it is done, set up the local kubeconfig:

    mkdir -p $HOME/.kube
    
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Verify that the cluster is responsive and that Kubectl is working:

    kubectl version

You should get Server Version as well as Client Version . It should look something like this:

The kubeadm init command should output a kubeadm join command containing a token and hash. Copy that command and run it with sudo on both worker nodes. It should look something like this:

    sudo kubeadm join $some_ip:6443 — token $some_token — discovery-token-ca-cert-hash $some_hash

Verify that all nodes have successfully joined the cluster:

    kubectl get nodes

You should see all three of your nodes listed.

**Note:** The nodes are expected to have a STATUS of NotReady at this point.

## K8s cluster networking using flannel

Once the Kubernetes cluster is set up, we still need to configure cluster networking in order to make the cluster fully functional.

We will walk through the process of configuring a cluster network using Flannel. You can find more information on Flannel at the official site: [https://coreos.com/flannel/docs/latest/](https://coreos.com/flannel/docs/latest/ "https://coreos.com/flannel/docs/latest/").

**Commands:**

On all three nodes, run the following:

    echo “net.bridge.bridge-nf-call-iptables=1” | sudo tee -a /etc/sysctl.conf
    
    sudo sysctl -p
    
    kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    
    kubectl get nodes

**Note:** It may take a few moments for all nodes to enter the Ready status, so if they are not all Ready , wait a few moments and try again.

It is also a good idea to verify that the Flannel pods are up and running. Run this command to get a list of system pods:

    kubectl get pods -n kube-system

You should have three pods with flannel in the name, and all three should have a status of Running.

## Deploy sample microservices app on K8s cluster

Kubernetes is a powerful tool for managing and deploying microservice applications.

Now we will deploy a microservice application consisting of multiple varied components to our cluster.

We will also explore the application briefly in order to get a hands-on glimpse of what a microservice application might look like, and how it might run in a Kubernetes cluster.

Here are the **commands** used in the demonstration to deploy the Stan’s Robot Shop application:

Clone the Git repository:

    cd ~/git clone https://github.com/linuxacademy/robot-shop.git

Create a namespace and deploy the application objects to the namespace using the deployment descriptors from the Git repository:

    kubectl create namespace robot-shop
    
    kubectl -n robot-shop create -f ~/robot-shop/K8s/descriptors/

Get a list of the application’s pods and wait for all of them to finish starting up:

    kubectl get pods -n robot-shop -w

Once all the pods are up, you can access the application in a browser using the public IP of one of your Kubernetes servers and port 30080:

    http://$kube_server_public_ip:30080