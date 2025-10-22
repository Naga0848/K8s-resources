Look at the K8s.png image to understand how kubectl, eksctl and docker works
We will have a workstation 
    - Docker for the images
    - Kubectl to interact with cluster via AWS CLI
    - eksctl to install the kubernetes cluster

# inorder to create k8s resources for roboshop project
    please use the below images from joindevops

        joindevops/cart
        joindevops/catalogue
        joindevops/mysql
        joindevops/shipping
        joindevops/payment
        joindevops/user
        joindevops/frontend
        joindevops/mongodb



# Points to remember when creating the cluster
 When you create an IAM user, make sure you give adminaccess to the user and while doing aws configure, please mention the region aswell along with access-key and secret-key

work station
============
1. install docker
2. run aws configure
3. install eksctl for cluster creation
4. install kubectl for cluster interaction

# Steps to install K8s

# 1) create a t3.medium ec2 instance and install aws cli on it and login via cli

    aws configure
    Access Key ID: create via AWS IAM
    Secret Key ID: create via AWS IAM

# 2) Install Docker 

    sudo dnf -y install dnf-plugins-core
    sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
    sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    sudo systemctl restart docker
    sudo systemctl enable --now docker
    # add your normal user to docker group
    sudo usermod -aG docker ec2-user
    exit, after adding normal user and re-login and start using the docker commands without giving sudo

# 3) eksctl Installation  (it is a command line tool)

    ARCH=amd64
    PLATFORM=$(uname -s)_$ARCH
    curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
    curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
    tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
    sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
    eksctl version

# 4) Kubectl installation

    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.0/2025-05-01/bin/linux/amd64/kubectl
    chmod +x ./kubectl   >> execution permission
    sudo mv kubectl /usr/local/bin/kubectl   >>> who ever logins will get k8s access
    kubectl version


# Finally After insalling the eksctl and kubectl, we have to use eks.yaml file(managenodegroup) and create our eks cluster

    eksctl create cluster --config-file=eks.yaml

    eksctl delete cluster --config-file=eks.yaml
        # while deleteing a cluster, first delete the autoscaling group and then manually delete the EC2 instances and then finally delete the cluster.
        # delete the node groups inside EKS console.
        #  Delete the cluster now
    By default our ec2 nodes will be created in the default-vpc

    Here, we can login to the ec2 instances(nodes), but we cannot login to the control plane which we see in tghe AWS console. ANd it is managed by AWS itself.



# Please refer to K8s-Objects-Versions.png to know which resource will have the what apiVersion in K8s

=======================================================================================================


# Understanding PV, PVC and StorageClass

        EBS static provisioning
        =======================
            1. Install EBS drivers
            2. Provide access to EC2 instance through role, EBSCSIDriverPolicy
            3. create volume in same az as in EC2 instance
            4. create PV, create PVC, create pod with nodeSelector option

    PV --> cluster level
    PVC --> namespace level

    EKS admins control cluster level objects...

    as a roboshop devops engineer, you need a disk to be created for your application,

    we will raise a ticket for this. storage (GB), filesystem type(ext4), etc..it is approved by roboshop team lead/delivery lead. Storage team also checks this and it is approved their team leader....

    then storage team creates the disk...

    provide these disk details to EKS admin, then they create PV for us and tell us the name.

    PV --> K8 resource, physical representation of the actual EBS storage.
    PVC --> it is the claim done by pods to mount the storage  (here we create the PVC yaml file, and if we want this storage to be attached to a pod, then we mention the pod definition file in the same yaml file)

    StorageClass --> k8 object used to create the volume dynamically...
        This is a cluster level object. If we have a storageclass yaml file, we can directly create the PVC and POD definition files in the other yaml file
        Understand that StorageClass yaml file is different and PVC and POD definition files can together.



Kid --> Mother --> Father --> Wallet
Pod -->	 PVC -->	PV 	  --> Storage

Kid --> Mother --> UPI
Pod --> PVC

# Shipping Microservice

**This service handles product catalog and inventory updates.**

It connects to MongoDB and Redis for data persistence and caching.

**Important:** Ensure Docker is running before deploying.
