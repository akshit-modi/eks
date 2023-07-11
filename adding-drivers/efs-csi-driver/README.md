# Setup EFS CSI driver

1. Create EFS CSI Plugin IAM serviceaccount 
2. Install EFS driver using helm

## Create EBS CSI Plugin IAM serviceaccount 

```
# Create policy
aws iam create-policy \
    --policy-name AmazonEKS_EFS_CSI_Driver_Policy \ # Change with custom name
    --policy-document file://efs-custom-kms-key-policy.json

# Create service account with same policy
eksctl create iamserviceaccount \
    --name efs-csi-controller-sa \
    --namespace kube-system \
    --cluster <CLUSTER_NAME> \ # CLUSTER_NAME
    --attach-policy-arn arn:aws:iam::111122223333:policy/AmazonEKS_EFS_CSI_Driver_Policy \ 
    --approve \
    --region <REGION>
```

## Installing EFS driver using helm
```
# Add helm repo
helm repo add aws-efs-csi-driver https://kubernetes-sigs.github.io/aws-efs-csi-driver/

# Update repo
helm repo update aws-efs-csi-driver

# Install EFS driver using chart
helm upgrade -i aws-efs-csi-driver aws-efs-csi-driver/aws-efs-csi-driver \
    --namespace kube-system \
    --set image.repository=602401143452.dkr.ecr.<REGION_CODE>.amazonaws.com/eks/aws-efs-csi-driver \ # update region
    --set controller.serviceAccount.create=false \
    --set controller.serviceAccount.name=efs-csi-controller-sa

# If you don't have outbound access to the Internet, add the following arguments. (Optional)
    --set sidecars.livenessProbe.image.repository=602401143452.dkr.ecr.<REGION_CODE>.amazonaws.com/eks/livenessprobe \
    --set sidecars.nodeDriverRegistrar.image.repository=602401143452.dkr.ecr.<REGION_CODE>.amazonaws.com/eks/csi-node-driver-registrar \
    --set sidecars.csiProvisioner.image.repository=602401143452.dkr.ecr.<REGION_CODE>.amazonaws.com/eks/csi-provisioner

# To force the Amazon EFS driver to use FIPS for mounting the file system, add the following argument (Optional)
    --set useFips=true
```



vpc_id=$(aws eks describe-cluster \
    --name my-cluster \
    --query "cluster.resourcesVpcConfig.vpcId" \
    --output text)

cidr_range=$(aws ec2 describe-vpcs \
    --vpc-ids $vpc_id \
    --query "Vpcs[].CidrBlock" \
    --output text \
    --region region-code)

security_group_id=$(aws ec2 create-security-group \
    --group-name MyEfsSecurityGroup \
    --description "My EFS security group" \
    --vpc-id $vpc_id \
    --output text)

aws ec2 authorize-security-group-ingress \
    --group-id $security_group_id \
    --protocol tcp \
    --port 2049 \
    --cidr $cidr_range

file_system_id=$(aws efs create-file-system \
    --region region-code \
    --performance-mode generalPurpose \
    --query 'FileSystemId' \
    --output text)
