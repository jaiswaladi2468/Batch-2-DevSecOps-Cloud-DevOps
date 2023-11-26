## Creating Service Account


```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

## Create Role 


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

## Bind the role to service account


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

## Generate token using service account in the namespace

[Create Token](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/#:~:text=To%20create%20a%20non%2Dexpiring,with%20that%20generated%20token%20data.)

## Pipeline



```groovy
pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/jaiswaladi246/Ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('Sonar Analysis') {
            steps {
               withSonarQubeEnv('sonar'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=E-Kart \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=E-Kart '''
               }
            }
        }
        
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
                    dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs ."
            }
        }
        
         stage('Build Application') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploy Artifacts to Nexus') {
            steps {
                
                withMaven(globalMavenSettingsConfig: '49407cf1-974f-4569-af17-7b94572504c6') {
                         sh "mvn deploy -DskipTests=true"
                }
               
            }
        }
        
        stage('Docker Build & Tag Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh "docker build -t shopping-cart:dev -f docker/Dockerfile ."
                            sh " docker tag shopping-cart:dev adijaiswal/shopping-cart:dev"
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image adijaiswal/shopping-cart:dev"
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                            sh " docker push adijaiswal/shopping-cart:dev"
                    }
                }
            }
        }
        
        stage('Kubernetes'){
            steps{
                withKubeConfig(caCertificate: '''-----BEGIN CERTIFICATE-----
MIIC5zCCAc+gAwIBAgIBADANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwprdWJl
cm5ldGVzMB4XDTIzMDkxMTA2MzAxOFoXDTMzMDkwODA2MzAxOFowFTETMBEGA1UE
AxMKa3ViZXJuZXRlczCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAMCA
wufr5BxHNinqKU4TATJilR7oT6jmBQm3SciWWFxYkPCTvDGHdgb7PFUmT/sNkf4z
dBjTbAq8fB9yq5J2p3U7FINNPH8joB51ofTP+4vQnRjwNawn5POTzWbY9pmuZ2le
zCE5FpmatveJ4tAXehjf6LQO2v1a3iD4kojctiat1JM5tOycNkteJmgb2nOlOLwO
0jglz/5m/XlSu+DEBE3PNAL/o7edM5xR1ofrroNQ54W/dVfQwpfL7jfxxJ36rMna
qOEClgJZk9uS6OmJFR5drMcuybyZOrWojZaBBvC5LzIERBlBKfcUH5RfmKqVUlb6
Y1N8/cJWhnQhre8J9u0CAwEAAaNCMEAwDgYDVR0PAQH/BAQDAgKkMA8GA1UdEwEB
/wQFMAMBAf8wHQYDVR0OBBYEFNTtVjt47g0QG3Z17bUh+Xj8S1dGMA0GCSqGSIb3
DQEBCwUAA4IBAQCcOI/DN9tjI1oy+i5iAqdQoRDXqHmkdkBVPYUx1TZ0H8it4ZYz
Dz2CzxnUrsIpli7M/RiS/5ENtpIIGxqhAfXnjemkKjXEAa3hBCxEX3n7ReQsK8QH
2HtyRdlDJwod/e1zSS3WnPMRfKQqXmmD1vZLzzbvdl+LSUtwat2C8Z3tNQsZDMPG
vKwEuVYjIG3NSlgPLMVlbgIEgZq8sUYiYTHbiFO1WpeCczpTV0101BW+/kGWRsW+
Y1yDb1xQ/6TrRbxj4fh/oq2dNq8Jgv3U79/mB1EevJZQudkB0pO9n7aRRietccNV
ea4SsY9qI2mARJftAF2UbBgqMO6Ez5s5pPxF
-----END CERTIFICATE-----''', clusterName: '', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.1.206:6443') {
    
        sh "kubectl apply -f deploymentservice.yml"
        sh "kubectl get pods"   
        sh "kubectl get svc" 
    
    }
}
```
