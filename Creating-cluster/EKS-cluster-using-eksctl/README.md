# Create EKS Cluster & Node Groups

## Pre requisite
- AWS CLI [Install guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
- Kubectl CLI [Install guide](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- Eksctl CLI [Install guide](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html#installing-eksctl)

## Step 1 Create cluster
- It will take 15 to 20 minutes to create the Cluster Control Plane 
```
# Create Cluster
eksctl create cluster --name=<CLUSTER_NAME> \
                      --region=<REGION> \
                      --zones=<AZ_1>,<AZ_2> \
                      --without-nodegroup 

# Get List of clusters
eksctl get cluster                  
```

## Step 2: Create & Associate IAM OIDC Provider for our EKS Cluster
- To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create &  associate OIDC identity provider.
- To do so using `eksctl` we can use the  below command. 
- Use latest eksctl version
```                   
# Replace with region & cluster name
eksctl utils associate-iam-oidc-provider \
    --region <REGION> \
    --cluster <CLUSTER_NAME> \
    --approve
```

## Step 3: Create Node Group with additional Add-Ons in Public Subnets
- You can use existing key or create a new key to login inside EKS worker node.
- These add-ons will create the respective IAM policies for us automatically within our Node Group role.
 ```
# Create Public Node Group   
eksctl create nodegroup --cluster=<CLUSTER_NAME> \
                        --region=<REGION> \
                        --subnet-ids=<SUBNET_IDS> \ # provide subnet-ids separated by comma. 
                        --node-private-networking \ # (optional) # provide this when you are passing private subnet-ids.
                        --name=<NODEGROUP_NAME> \
                        --node-type=t3.medium \ # update node type
                        --nodes=1 \ 
                        --nodes-min=1 \
                        --nodes-max=2 \
                        --node-volume-size=20 \ # update volume
                        --ssh-access \
                        --ssh-public-key=kube-demo \ # update key name here
                        --managed \
                        --asg-access \
                        --external-dns-access \
                        --full-ecr-access \
                        --appmesh-access \
                        --alb-ingress-access 
```

## Step 4: Verify Cluster & Nodes

### Verify Cluster, NodeGroup in EKS Management Console
- Go to Services -> Elastic Kubernetes Service -> eksdemo1

```
# List EKS clusters
eksctl get cluster

# List NodeGroups in a cluster
eksctl get nodegroup --cluster=<CLUSTER_NAME>

# List Nodes in current kubernetes cluster
kubectl get nodes -o wide

# Our kubectl context should be automatically changed to new cluster
kubectl config view --minify
```

### Verify Worker Node IAM Role and list of Policies
- Go to Services -> EC2 -> Worker Nodes
- Click on **IAM Role associated to EC2 Worker Nodes**

### Verify Security Group Associated to Worker Nodes
- Go to Services -> EC2 -> Worker Nodes
- Click on **Security Group** associated to EC2 Instance which contains `remote` in the name.

## Notes: 
- If you stuck on any step and resource is not able tp create. Try to remove the same stack from cloud formation console and rerun the command. 
- If you are using different aws profile you can pass that as a argument in eksctl command. 