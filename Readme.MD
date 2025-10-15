Look at the K8s.png image to understand how kubectl, eksctl and docker works
We will have a workstation 
    - Docker for the images
    - Kubectl to interact with cluster via AWS CLI
    - eksctl to install the kubernetes cluster

# When you create an IAM user, make sure you give adminaccess to the user and while doing aws configure, please mention the region aswell along with access-key and secret-key

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
        # delete the node groups inside EKS. 
    By default our ec2 nodes will be created in the default-vpc

    Here, we can login to the ec2 instances(nodes), but we cannot login to the control plane which we see in tghe AWS console. ANd it is managed by AWS itself.



