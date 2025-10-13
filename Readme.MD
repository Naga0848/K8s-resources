
Look at the K8s.png image to understand how kubectl, eksctl and docker works
We will have a workstation 
    - Docker for the images
    - Kubectl to interact with cluster via AWS CLI
    - eksctl to install the kubernetes cluster


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

    PLATFORM=$(uname -s)_$ARCH
    curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
    curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
    tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
    sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
    eksctl version

# 4) Kubectl installation

    