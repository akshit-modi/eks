# Setup EBS CSI driver

1. Create EBS CSI Plugin IAM serviceaccount 
2. Add `Amazon EBS CSI Driver` add-on through EKS console 



## Create EBS CSI Plugin IAM serviceaccount 
```
  eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster <CLUSTER_NAME> \ # CLUSTER_NAME
    --role-name <AmazonEKS_EBS_CSI_DriverRole> \  # change with custom role name.
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```

- (optional) if you want to use your own kms key for EBS encryption, 
  ```
  # Update the key arn in ebs-custom-eks-key-policy.json file. 
  
  # Create policy using json file
  aws iam create-policy \
  --policy-name KMS_Key_For_Encryption_On_EBS_Policy \ # policy name
  --policy-document file://ebs-custom-eks-key-policy.json # json file name

  # Attach policy to role
  aws iam attach-role-policy \
  --policy-arn arn:aws:iam::111122223333:policy/KMS_Key_For_Encryption_On_EBS_Policy \ # above mentioned policy name
  --role-name AmazonEKS_EBS_CSI_DriverRole # above created role
  ```

2. Add EBS csi driver add-on using the aws management console. 
  - While choosing the add-on, in role name, use same role which was created above. 

