## ingress Controller

REGION_CODE=us-east-1
CLUSTER_NAME=expense
ACC_ID=180935779261

### Permissions

to connect aws to our expense cluster with authentication service

1. Create an IAM OIDC provider. You can skip this step if you already have one for your cluster. --->OIDC Provider --- Admin Task

...

eksctl utils associate-iam-oidc-provider \
    --region $REGION_CODE \
    --cluster $CLUSTER_NAME \
    --approve
...

2. Download an IAM policy for the LBC using one of the following commands: ---> IAM Policy

curl -o iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.17.0/docs/install/iam_policy.json

3. Create an IAM policy named AWSLoadBalancerControllerIAMPolicy. If you downloaded a different policy, replace iam-policy with the name of the policy that you downloaded. ---> Create IAM role

...

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam-policy.json

...

...
"Policy": {
        "PolicyName": "AWSLoadBalancerControllerIAMPolicy",
        "PolicyId": "ANPASUIE44O6VRFAWZROL",
        "Arn": "arn:aws:iam::180935779261:policy/AWSLoadBalancerControllerIAMPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2026-01-04T06:32:56+00:00",
        "UpdateDate": "2026-01-04T06:32:56+00:00"
...

4. Create an IAM role and Kubernetes ServiceAccount for the LBC. Use the ARN from the previous step. ---> Provide access to EKS through IAM Policy

...

eksctl create iamserviceaccount \
--cluster=$CLUSTER_NAME \
--namespace=kube-system \
--name=aws-load-balancer-controller \
--attach-policy-arn=arn:aws:iam::$ACC_ID:policy/AWSLoadBalancerControllerIAMPolicy \
--override-existing-serviceaccounts \
--region $REGION_CODE \
--approve

...


### Install Drivers

* Add the EKS chart repo to Helm
...

helm repo add eks https://aws.github.io/eks-charts

...

* Install AWS Load Balancer Controller
...

helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$CLUSTER_NAME --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller

...

eksctl create iamserviceaccount \
--cluster=expense \
--namespace=expense \
--name=expense-mysql-secret \
--attach-policy-arn=arn:aws:iam::180935779261:policy/ExpenseMySQLSecrectRead \
--override-existing-serviceaccounts \
--region us-east-1 \
--approve

