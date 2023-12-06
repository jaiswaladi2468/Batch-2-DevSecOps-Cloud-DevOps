## First Create a user in AWS IAM with any name
## Attach Policies to the newly created user
## below policies
AmazonEC2FullAccess

AmazonEKS_CNI_Policy

AmazonEKSClusterPolicy	

AmazonEKSWorkerNodePolicy

AWSCloudFormationFullAccess

IAMFullAccess

#### One more policy we need to create with content as below
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": "eks:*",
            "Resource": "*"
        }
    ]
}
```
Attach this policy to your user as well

![Policies To Attach](https://github.com/jaiswaladi2468/Batch-2-DevSecOps-Cloud-DevOps/blob/main/12.%20Kubernetes/EKS%2BJENKINS/Policies.png)

# AWSCLI

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
aws configure
```

## KUBECTL

```bash
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

## EKSCTL

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

## Create EKS CLUSTER

```bash
eksctl create cluster --name=my-eks2 \
                      --region=ap-south-1 \
                      --zones=ap-south-1a,ap-south-1b \
                      --without-nodegroup

eksctl utils associate-iam-oidc-provider \
    --region ap-south-1 \
    --cluster my-eks2 \
    --approve

eksctl create nodegroup --cluster=my-eks2 \
                       --region=ap-south-1 \
                       --name=node2 \
                       --node-type=t3.medium \
                       --nodes=3 \
                       --nodes-min=2 \
                       --nodes-max=3 \
                       --node-volume-size=20 \
                       --ssh-access \
                       --ssh-public-key=Key \
                       --managed \
                       --asg-access \
                       --external-dns-access \
                       --full-ecr-access \
                       --appmesh-access \
                       --alb-ingress-access
```

* Open INBOUND TRAFFIC IN ADDITIONAL Security Group
* Create Servcie account/ROLE/BIND-ROLE/Token [link](https://github.com/jaiswaladi2468/Batch-2-DevSecOps-Cloud-DevOps/blob/main/12.%20Kubernetes/Kubernetes-Jenkins.md)

## Create Service Account, Role & Assign that role, And create a secret for Service Account and geenrate a Token

### Creating Service Account


```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

### Create Role 


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

### Bind the role to service account


```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role 
subjects:
- namespace: webapps 
  kind: ServiceAccount
  name: jenkins 
```

### Generate token using service account in the namespace

[Create Token](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#:~:text=To%20create%20a%20non%2Dexpiring,with%20that%20generated%20token%20data.)

## Docker Install on VM

```bash
sudo apt install docker.io
sudo chmod 666 /var/run/docker.sock
```

## Jenkins Install Plugins

* kubernetes cli
* kubernetes
* Docker
* CloudBees Docker Build and Publish plugin
* docker-build-step
* Docker Pipeline

# JENKINS PIPELINE

```groovy
pipeline {
    agent any

    stages {
        stage('Git') {
            steps {
                git branch: 'latest', url: 'https://github.com/jaiswaladi246/10-Tier-MicroService-Appliction.git'
            }
        }
        
        
        stage('adservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/adservice/') {
                                 sh "docker build -t adijaiswal/adservice:latest ."
                                 sh "docker push adijaiswal/adservice:latest"
                        }
                    }
                }
            }
        }
		
		stage('cartservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/cartservice/src/') {
                                 sh "docker build -t adijaiswal/cartservice:latest ."
                                 sh "docker push adijaiswal/cartservice:latest"
                        }
                    }
                }
            }
        }
		
		stage('checkoutservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/checkoutservice/') {
                                 sh "docker build -t adijaiswal/checkoutservice:latest ."
                                 sh "docker push adijaiswal/checkoutservice:latest"
                        }
                    }
                }
            }
        }
		
		stage('currencyservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/currencyservice/') {
                                 sh "docker build -t adijaiswal/currencyservice:latest ."
                                 sh "docker push adijaiswal/currencyservice:latest"
                        }
                    }
                }
            }
        }
        
		stage('emailservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/emailservice/') {
                                 sh "docker build -t adijaiswal/emailservice:latest ."
                                 sh "docker push adijaiswal/emailservice:latest"
                        }
                    }
                }
            }
        }
		
		stage('frontend') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/frontend/') {
                                 sh "docker build -t adijaiswal/frontend:latest ."
                                 sh "docker push adijaiswal/frontend:latest"
                        }
                    }
                }
            }
        }
		
		stage('loadgenerator') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/loadgenerator/') {
                                 sh "docker build -t adijaiswal/loadgenerator:latest ."
                                 sh "docker push adijaiswal/loadgenerator:latest"
                        }
                    }
                }
            }
        }
		
		stage('paymentservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/paymentservice/') {
                                 sh "docker build -t adijaiswal/paymentservice:latest ."
                                 sh "docker push adijaiswal/paymentservice:latest"
                        }
                    }
                }
            }
        }
        
		stage('productcatalogservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/productcatalogservice/') {
                                 sh "docker build -t adijaiswal/productcatalogservice:latest ."
                                 sh "docker push adijaiswal/productcatalogservice:latest"
                        }
                    }
                }
            }
        }
		
		stage('recommendationservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/recommendationservice/') {
                                 sh "docker build -t adijaiswal/recommendationservice:latest ."
                                 sh "docker push adijaiswal/recommendationservice:latest"
                        }
                    }
                }
            }
        }
		
		stage('shippingservice') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                          dir('/var/lib/jenkins/workspace/Test/src/shippingservice/') {
                                 sh "docker build -t adijaiswal/shippingservice:latest ."
                                 sh "docker push adijaiswal/shippingservice:latest"
                        }
                    }
                }
            }
        }
        
        stage('K8') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'my-eks', contextName: '', credentialsId: '40662307-ca84-4893-ab4b-f6ffadfc3f8c', namespace: 'devops', restrictKubeConfigAccess: false, serverUrl: 'https://7DF03B658F34713AF60204C79AB4DF84.gr7.ap-south-1.eks.amazonaws.com') {
                       sh "kubectl apply -f deployment-service.yml"
					   sh "kubectl get pods -n devops"
					   sh "kubectl get svc -n devops"
}
            }
        }


    }
}
```
