#### This repo contains different kubernetes resources and their example yaml files

Look at the K8s.png image to understand how kubectl, eksctl and docker works
We will have a workstation i.e., using which we install the Docker, eksctl, kubectl, and aws configure
    - Docker for the images
    - Kubectl to interact with cluster via AWS CLI
    - eksctl to install the kubernetes cluster

#### inorder to create k8s resources for roboshop project
    please use the below images from joindevops

        joindevops/cart
        joindevops/catalogue
        joindevops/mysql
        joindevops/shipping
        joindevops/payment
        joindevops/user
        joindevops/frontend
        joindevops/mongodb



#### Points to remember when creating the cluster
 When you create an IAM user, make sure you give adminaccess to the user and while doing aws configure, please mention the region aswell along with access-key and secret-key

#### On a work station
    1. install docker
    2. run aws configure
    3. install eksctl for cluster creation
    4. install kubectl for cluster interaction

#### Steps to install K8s

#### 1) create a t3.medium ec2 instance and install aws cli on it and login via cli

    aws configure
    Access Key ID: create via AWS IAM
    Secret Key ID: create via AWS IAM

#### 2) Install Docker 

    sudo dnf -y install dnf-plugins-core
    sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
    sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    sudo systemctl restart docker
    sudo systemctl enable --now docker
    # add your normal user to docker group
    sudo usermod -aG docker ec2-user
    exit, after adding normal user and re-login and start using the docker commands without giving sudo

#### 3) eksctl Installation  (it is a command line tool)

    ARCH=amd64
    PLATFORM=$(uname -s)_$ARCH
    curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
    curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
    tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
    sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
    eksctl version

#### 4) Kubectl installation

    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.0/2025-05-01/bin/linux/amd64/kubectl
    chmod +x ./kubectl   >> execution permission
    sudo mv kubectl /usr/local/bin/kubectl   >>> who ever logins will get k8s access
    kubectl version


#### Finally After installing the eksctl and kubectl, we have to use eks.yaml file(managenodegroup) and create our eks cluster

    eksctl create cluster --config-file=eks.yaml

    eksctl delete cluster --config-file=eks.yaml
        # while deleting a cluster, first delete the autoscaling group and then manually delete the EC2 instances and then finally delete the cluster.
        # delete the node groups inside EKS console.
        #  Delete the cluster now
    By default our ec2 nodes will be created in the default-vpc

    Here, we can login to the ec2 instances(nodes), but we cannot login to the control plane which we see in tghe AWS console. ANd it is managed by AWS itself.

#### Please refer to K8s-Objects-Versions.png to know which resource will have the what apiVersion in K8s


#### Volumes
#### Understanding PV, PVC and StorageClass (Elastic Block Storage)EBS and (Elastic File System)EFS

    EBS static provisioning
    =======================
        1. Install EBS drivers  kubectl apply -k "github.com/kubernetes-sigs/aws-ebs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.43  (should be installed on EKS)
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

#### If we attach the policy to a single node, it gets attached to all the 3 nodes, because we are using same IAM role across all the nodes
        EC2 >>> Security >> IAM Roles >> Add Permission >> Add the required policy
    
    EFS static provisioning
    =======================
        1. Install drivers  
            kubectl kustomize \
    "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-2.1" > private-ecr-driver.yaml
        2. Give permissions to nodes, EFSCSIDriverPolicy
        3. create EFS volume
        4. Allow port no 2049 in EFS SG from EC2 SG  (Go to SG of EFS > NFS, 2049 and now give the SG of EC2 nodes rather than giving IP and save the rules)
        5. Create PV, PVC and mount to pod

    EBS vs EFS
    ===========
    1. EFS can be anywhere in network, EBS should be in same AZ as in EC2
    2. EBS can be accessed only by one instance at a time, EFS can be accessed by multiple instances
    3. EBS is faster than EFS.
    4. EBS is mainly used for OS and DB, EFS is used to store objects and files.
    5. EBS size is fixed, EFS is elastic it automatically grows upto 48TB.
    6. EBS filesystem we can select ext4, EFS filesystem is already decided by AWS as NFS
    7. EBS will not have any SG attached, but EFS is in network, so there will be SG attached and 2049 should be allowed.


#### Understanding Commands

    First we have to run the pv command i.e., kubectl apply for a PV
        Here u can see that a PV or EBS is created
    Second we have to run the pvc command i.e., kubectl apply for a PVC
        Here u can see that EBS is mounted to the POD, if we mentioned the pod definition file in the yaml file

#### Things to be remembered while creating EFS AND EBS

    when creating statically i.e., manually
        we need to create PV, PVC, POD and service(LB) in the same yaml file

        PV = physical storage = EBS Volume  (its a cluster level object)
        PVC = we are using this to claim a volume for our pod
        service = when we attached we volume to our pod, we have to access it right, as a result we are using the LB. And once everything got created, please take the LB created and hit it in the browser and we get 403 Forbiden error as we dont have anything inside our volume
            Now, go to the location where your entire yaml file is loacted and perform the below commands to create a html file
                kubect exec -it nginx -- bash
                cd /usr/share/nginx/html 
                "<h1> Hi, i am from EBS Static </h1>" > index.html
                And when u hit the same browser with LB, we can see the ablove text.
                And if u try deleting the pod, we cannot acces the LB. gain, if  we create the pod, the EBS gets attached to the same pod
                Same happens with the EFSas well when provisioning statically

        Understand that we have to use the nodeselector if we want to place a pod on the specific Node
            For this, we have to see the labels which are on node  (kubectl get nodes -- labels)
            From this we have to look for a label with region and use the same label as a selector in the pod definition file

        When provisioning EBS Dynamically
            First create the StorageClass 
            And the PVC, POD and Service(LB) should be in a separate yaml file
            Here, storageClass will take care of creating the PV for us.
            Here Volume will be created by EKS ADMIN Team and we will make use of it by using our StorageClass
            The StorageClass is one time creation
            In PVC definition file, we have to mention the storageclass name and the disk will be attached to the desired pod.

        Steps for deleting
            Delete the PVC
            Delete PV
            Delete POD
            And finally the Volume(EBS or EFS)

        When provisioning EFS Dynamically (Here access points gets created, which are like EFS)the name of the accesspoint and PV will be same(please check)
            First create the StorageClass
            Create PVC, POD and Service in the different yaml file
            And finally, DATA will be accessed on the access points


        
#### Understanding  About Statefulsets

    stateful applications --> DB

    Deployment --> stateless applications not for stateful applications
    Statefulset --> stateful applications.

    Deployment vs Statefulset
    ==========================
    1. Deployment is for stateless applications, Statefulset is for stateful applications
    2. PV and PVC is not mandatory for Deployment, but mandatory for stateful applications
    3. Statefulset need headless service... i.e no cluster IP
    4. Pods create in orderly manner in statefulset, Once first pod comes to running, then only other pod will create. While deletion reverse order follows
    5. Pod identities are preserved in statefulset, because if any pod crashes, statefulset create another pod with same name, so that communication is easy between pods..


#### Now its time for understanding the different k8s resources and their yaml file syntax

#### 1) namespace.yaml 
    
    A way to organize and isolate groups of resources in a cluster.

    apiVersion: v1
    kind: Namespace
    metadata: 
        name: roboshop
        labels:
            name: robosh
            env: dev-env

#### 2) pod.yaml

    The smallest deployable unit that holds one or more containers i.e., everything in kubernetes is a pod

    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx
    spec:
        containers:
        - name: nginx
          image: nginx

#### 3) Multi-container Pod	

    A pod running multiple containers can have the same container or the other containers....

    apiVersion: v1
    kind: Pod
    metadata:
        name: multi-containe
        labels:  
            project: roboshop
            env: dev-env
    spec:
        containers:
        - name: nginx
          image: nginx
        - name: almalinux
          image: almalinux:9
    
    kubectl apply -f 03-multicontainer.yaml   >>> command to run the multicontainer pod

    here when we try to create a pod, we get crashloopbackof error for the second image, as we dont have any deamon or command to run the almalinux 
    continuosly. Or we should have some application inside almalinux to keep it running continuosly.In this case we are using a sleep command CMD ["sleep","1000s"]in the dockerfile itself to keep the almalinux live upto 1000s and then terminate.

    Sometimes even though we get the error, in that case we have to directly delete the pod and then re-create a new one.

    kubectl exec -it multicontainer -c almalinux --bash (to login to the second container) within the multicontainer pod.

    curl localhost

    even after logging into to the second container, we could still see the nginx welcome page. This is because the nginx and the almalinux share the same 
    network.

    multi containers are useful in side car patterns and proxy patterns. proxy means first proxy container gets the request, it checks whether request should be forwarded to main container or not. And sidecar container means, it will collect all the logs from the first contaner(as they share same storage which canbe accessed by sidecar as well) and then send them to ELK(external for storing logs).

    catalogue, user, cart, shipping, payment, frontend can be in same pod as multiple containers but diff ports should be opened by containers....so using 
    different container in the same pod is not a good approach in orchestration.

#### 4) Labels 

    Key-value tags used to identify and group resources kubernetes 

    apiVersion: v1
    kind: Pod
    metadata:
        name: labels
        labels:
            project: rob
            env: dev-env
            tier: fronte
    spec:
        containers:
        - name: nginx
          image: nginx

    Lables are used as selectors for other resources inside kuberentes... useful for filtering resources. lables values we can't keep long values, only 63char

#### 5) Annotations

    Annotations in Kubernetes are similar to labels, it is a ke-value pair but not used as selectors for identifying other resources. Annotations are used for provisioning the external resources to k8s. We keep external information like build url, image registery etc..
    Annotations values can be upto 256char
    apiVersion: v1
    kind: Pod
    metadata:
        name: annotati
        labels:
            project: rob
            component: m
            tier: backen
            environment: dev
        annotations:
            description: "This pod is created to demonstrate pod annotations"
            jenkins: "https://jenkins.com/build/job/roboshop-catalouge/3"
    spec:
        containers:
        - name: nginx
          image: nginx

#### 6) Resource Limits	

    Controls how much CPU and memory a container can use.

    apiVersion: v1
    kind: Pod
    metadata:
        name: resource-limits
        labels:
            env: dev-env
            tier: frontend
    spec:
        containers:
        - name: nginx
          image: nginx
        resources:
            requests: # soft limit
                memory: "68Mi"
                cpu: "100m"
            limits: # hard limit  (cannot extend beyond this)
                memory: "128Mi"
                cpu: "150m"


    1 processor = 1000m, 250m = 0.25

    kubectl top pods >>> command to see which pod specific resources

    After doing a performance testing either archetict or the developers will confirm the resources limits to us

    Understanding Horizontal Auto scaling and Vertical Auto scaling in a simple words

    Vertical auto scaling
        Understanding vertical scaling with an example
        In general if u want to construct a firstfloor and secondfloor on the existing 20years old building
        - you shouldn't stay in the house while remodelling (downtime) in our case, when u r increasing the resources manually, the application inside our container will not be available to the users
        - your basement may not be strong to bear another 2 floors
        ratherthan doing this u can take new land and construct two new houses(autoscaling)
        so, we can understand that in VA, increasing the resouces limits from 2GB to 4GB in the same server

    Horizontal auto scaling
      rather than going to VA - companies prefer to go for the HA.
      increasing the no of servers with te same resources i.e., adding two more servers of 2GB

#### 7) Environment Variables	

    When you create a Pod, you can set environment variables for the containers that run in the Pod. To set environment variables, include the env or envFrom field in the configuration file.

    The env and envFrom fields have different effects.
    env -- allows you to set environment variables for a container, specifying a value directly for each variable that you name.
    envFrom -- allows you to set environment variables for a container by referencing either a ConfigMap or a Secret. When you use envFrom, all the key-value pairs in the referenced ConfigMap or Secret are set as environment variables for the container. You can also specify a common prefix string.

    apiVersion: v1
    kind: Pod
    metadata:
        name: envar-demo
        labels:
        purpose: demonstrate-envars
    spec:
        containers:
        - name: nginx
          image: nginx
          env:
          - name: COURSE
            value: "KUBERENETS"
          - name: TRAINER
            value: "SIVAKUMAR REDDY M"

    Steps to execute the above yaml file

    kubect apply -f 07-env.yaml

    kubectl exec -it envar-demo -- bash

    env

    After executing (kubectl exec -it envar-demo -- bash) and (env) commands, we can see the environmen

    exit --- to come out of the pod

    code and configuration. try to keep configuration always outside(loose coupling)

    we try to keep the configuration outside of pod definition. i.e configmap

    And finally, we keep env variables and the configs like db related in a configmap and we refer them inside the pod.

#### 8) ConfigMap and 9) Pod with ConfigMap

    Stores non-sensitive configuration data as key-value pairs.

    When you create a Pod, you can set environment variables for the containers that run in the Pod. To set environment variables, include the env or 
    envFrom field in the configuration file.

    The env and envFrom fields have different effects.
    env -- allows you to set environment variables for a container, specifying a value directly for each variable that you name.
    envFrom -- allows you to set environment variables for a container by referencing either a ConfigMap or a Secret. When you use envFrom, all the 
    key-value pairs in the referenced ConfigMap or Secret are set as environment variables for the container. You can also specify a common prefix string.

    apiVersion: v1
    kind: ConfigMap
    metadata:
        name: pod-config
        labels:
            purpose: config
            env: dev
    data:
        COURSE: "DevOps with AWS"
        DURATION: "120HRS"
        TRAINER: "SIVAKUMAR REDDY M"

    kubectl apply -f 08-configmap.yaml 
    kubectl get configmaps

    now u can see the configmap with the name pod-config and we have to attach this or refer to a pod, so for that we have to write a pod definition file
    And we have to refer the configmap in a pod using envfrom

    Below is the Pod with ConfigMap	=== A pod that loads config values from a ConfigMap.

    apiVersion: v1
    kind: Pod
    metadata:
        name: env-configmap
    spec:
        containers:
        - name: nginx
          image: nginx
          envFrom:
            - configMapRef:
                name: pod-config   # name of the configmap i.e., 08-configmap.yaml

    Steps to execute the above yaml file
    
    kubectl apply -f 09-pod-config.yaml
    kubectl get pods
    kubect exec -it env-configmap -- bash
    env
    Now, if we execute the line no (kubect exec -it env-configmap -- bash) and (env), we will be inside the pod(env-configmap) and we can see our 
    configmap details, which we mentioned in our 08-configmap.yaml
    exit, to come out of the pod

#### 10) secret and 11) Pod with Secret

    Secret -- Stores sensitive data like passwords or tokens securely.

    apiVersion: v1
    kind: Secret
    metadata:
        name: pod-secret
        labels:
        purpose: secret
        env: dev
    type: Opaque   # as we are defining the data
    data:
        username: "YWRtaW4="       # this is from encoding
        password: "YWRtaW4zMjE="   # this is from encoding

    please refer to the base64-encoding.png to understand how we encoded the username and passsword
    kubectl apply -f 10-secret.yaml
    kubectl get pods
       now u can see the secret with the name pod-secret and we have to attach this to a pod, so for that we have to write a pod definition file 

    Pod with Secret ---- A pod that uses secret values for secure configuration.

    apiVersion: v1
    kind: Pod
    metadata:
        name: env-secret
    spec:
        containers:
        - name: nginx
          image: nginx
            envFrom:
            - secretRef:
                name: pod-secret

    here are we are refering the secrets using envFrom

    Steps to execute the above yaml files

    kubectl apply -f 11-pod-secret.yaml
    kubectl get pods
    kubectl exec -it env-secret -- bash 
    env
    now we can see the username and password which we mentioned in 10-secret.yaml
    And this is not say way to mention the secrets here. Please refer, base64-decoding.png to know how we can decode the password
    So, we have to encrypt the screts, not encoding
    encrypting means publickkey and privatekeys i.e., who ever has both the keys only can view the password. where as encoding means, who ever knows the 
    encoded value, they can decrypt and view the password and can miss use it.

#### 12) Service

    For some parts of your application (for example, frontends) you may want to expose a Service onto an external IP address, one that's accessible from 
    outside of your cluster i.e., exposes pods to other services or external traffic.

    Here we are using both pod and service in a single yaml file for better understanding.

    If we dont mention any type of service, it is considered as ClusterIP service

    apiVersion: v1
    kind: Service
    metadata:
        name: nginx
        labels:
        purpose: service-demo
    spec:
        selector:
            purpose: service-demo
            project: roboshop
            environment: dev
        ports:
          - protocol: TCP
            port: 80 # service port
            targetPort: 80 # container port
----------
    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx
        labels:
            purpose: service-demo
            project: roboshop
            environment: dev
    spec:
        containers:
        - name: nginx
          image: nginx

    In gereral, there are three types of services in kubernetes
    ClusterIP
    Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a type for a Service. You can expose the Service to the public internet using an Ingress or a Gateway.
    NodePort
    Exposes the Service on each Node's IP at a static port (the NodePort). To make the node port available, Kubernetes sets up a cluster IP address, the same as if you had requested a Service of type: ClusterIP.
    LoadBalancer
    Exposes the Service externally using an external load balancer. Kubernetes does not directly offer a load balancing component; you must provide one, or you can integrate your Kubernetes cluster with a cloud provider.

   NodePort Service --- Makes a pod accessible on a static port of each node.

   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-np
     labels:
        purpose: service-np-demo
   spec:
   type: NodePort
   selector:
     purpose: service-np-demo
     project: roboshop
     environment: dev
   ports:
     - protocol: TCP
       port: 80 # service port
       targetPort: 80 # container port
--
    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx-np
        labels:
            purpose: service-np-demo
            project: roboshop
            environment: dev
    spec:
        containers:
        - name: nginx
          image: nginx
    
    LoadBalancer Service ---- Exposes a pod to the internet via a cloud load balancer.

    apiVersion: v1
    kind: Service
    metadata:
        name: nginx-lb
        labels:
            purpose: service-lb-demo
    spec:
        type: LoadBalancer
        selector:
            purpose: service-lb-demo
            project: roboshop
            environment: dev
        ports:
          - protocol: TCP
            port: 80 # service port
            targetPort: 80 # container port
----------------------
    apiVersion: v1
    kind: Pod
    metadata:
        name: nginx-lb
        labels:
            purpose: service-lb-demo
            project: roboshop
            environment: dev
    spec:
        containers:
        - name: nginx
          image: nginx
