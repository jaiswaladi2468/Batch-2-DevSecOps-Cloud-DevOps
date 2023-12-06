# Setup Ingress

### Step 1: Create IAM Policy
```bash
mkdir folder && cd folder

curl -o iam_policy_latest.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy_latest.json
```
This script downloads the IAM policy JSON file for the AWS Load Balancer Controller and then creates an IAM policy named `AWSLoadBalancerControllerIAMPolicy` using that JSON file.

### Step 2: Create IAM Service Account
```bash
eksctl create iamserviceaccount \
  --cluster=my-eks5 \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::<Account-ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve
```
This script creates an IAM service account named `aws-load-balancer-controller` in the `kube-system` namespace of the EKS cluster. It attaches the IAM policy created in the previous step to this service account.

### Step 3: Install Helm
Follow the Helm installation guide: [Helm Installation](https://helm.sh/docs/intro/install/)

### Step 4: Install AWS Load Balancer Controller using Helm
```bash
helm upgrade --install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-eks5 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=ap-south-1 \
  --set vpcId=vpc-0dfe95ede3b0a9984
```
This Helm command installs the AWS Load Balancer Controller on the EKS cluster. It specifies configuration options such as the cluster name, service account settings, AWS region, and VPC ID.

### Step 5: Create Ingress Class
```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: my-aws-ingress-class
  annotations:
    ingressclass.kubernetes.io/is-default-class: "true"
spec:
  controller: ingress.k8s.aws/alb
```
This YAML file defines an IngressClass resource named `my-aws-ingress-class` with annotations specifying it as the default class for ALB.

### Step 6: Create Ingress Rules
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-cpr-demo
  annotations:
    alb.ingress.kubernetes.io/load-balancer-name: cpr-ingress
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP 
    alb.ingress.kubernetes.io/healthcheck-port: traffic-port
    alb.ingress.kubernetes.io/healthcheck-interval-seconds: '15'
    alb.ingress.kubernetes.io/healthcheck-timeout-seconds: '5'
    alb.ingress.kubernetes.io/success-codes: '200'
    alb.ingress.kubernetes.io/healthy-threshold-count: '2'
    alb.ingress.kubernetes.io/unhealthy-threshold-count: '2'   
spec:
  ingressClassName: my-aws-ingress-class
  rules:
    - http:
        paths:           
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port: 
                  number: 80
```
This YAML file defines an Ingress resource named `ingress-cpr-demo` with annotations specifying ALB settings. It also specifies an IngressClass reference (`my-aws-ingress-class`). The Ingress rules forward traffic to the `frontend` service on port 80 for requests to the root path.

