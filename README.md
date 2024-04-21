## Deploy-to-EKS-Cluster-from-Jenkins-Pipeline

### Steps to Configure

1. **Install kubectl on Jenkins Server**

    - SSH to the Jenkins Docker container:
    
      ```bash
      docker exec -u 0 -it <container_id> bash
      ```
      
    - Download and install kubectl:
    
      ```bash
      curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
      chmod +x ./kubectl
      mv ./kubectl /usr/local/bin/kubectl
      ```
      
    - Verify kubectl installation:
    
      ```bash
      kubectl version
      ```

2. **Install AWS IAM Authenticator**

    ```bash
    curl -Lo aws-iam-authenticator https://github.com/kubernetes-sigs/aws-iam-authenticator/releases/download/v0.6.11/aws-iam-authenticator_0.6.11_linux_amd64
    chmod +x ./aws-iam-authenticator
    mv ./aws-iam-authenticator /usr/local/bin
    ```

3. **Create kubeconfig file for Jenkins**

    Create a file named `config` and add the following Kubernetes configuration:
    
    ```yaml
    apiVersion: v1
    kind: Config
    clusters:
    - cluster:
        certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURCVENDQWUyZ0F3SUJBZ0lJWTRQMFVycTZ3emt3RFFZSktvWklodmNOQVFFTEJRQXdGVEVUTUJFR0ExVUUKQXhNS2EzVmlaWEp1WlhSbGN6QWVGdzB5TkRBME1qRXhNakV5TkRkYUZ3MHpOREEwTVRreE1qRTNORGRhTUJVeApFekFSQmdOVkJBTVRDbXQxWW1WeWJtVjBaWE13Z2dFaU1BMEdDU3FHU0liM0RRRUJBUVVBQTRJQkR3QXdnZ0VLCkFvSUJBUUNxYkRmSkkxMUhaMmF0UWhXVHBBbmtXeDM1bW5XWkZJUjQ4MzREU3BiSVFBTTJyK0Q4U3V2cUNRQ0YKVEVENTVrR09RR2UvZkhZalFtUFROa0xUYzIwMEdRRGJ1bDZ4YmZ6TzFXWHd5cEhyQVRnbU4wWXBCbTljckJ4MQpVZXY1Y3ZmL3BSNGNWNWpNQ0pRWEdZVUlGT1BYUXdQNjhiNml3bDBPc0ZranZNUm5jck1vSGxPVDhJTEQrYithCk9PbzZiZEhISVpsRDduYnlkZERvMnJEcDNyQTVjd1dVL25vWEdMUy95TDAyMVBGS3VqN2tTUUNZaFVKVnRUeDMKcTRONEpFUWRMckFFekdnamNucElTbXREUTEwZ3RGcnFGendOeVZKbXVwbU55MnpZTWE3YW41cTQ2ZmdHRzUwSgp3YnVBV0c1Mk5Ld2RaRGZCbzRmMk85dzVISXlQQWdNQkFBR2pXVEJYTUE0R0ExVWREd0VCL3dRRUF3SUNwREFQCkJnTlZIUk1CQWY4RUJUQURBUUgvTUIwR0ExVWREZ1FXQkJRWGVCZ1VoZSt3eUJFcXd6dXg0QU85RlE0ek5qQVYKQmdOVkhSRUVEakFNZ2dwcmRXSmxjbTVsZEdWek1BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQWVLdnBTOXFucgpEL0R1UnQ1ZTlvTC95dyt1RXJqVngrUXduYzNnaDJkc2MyNE44ZTlFaTI2eDdhMnJib01xRkNLa2dZZ2pielQyCkhkUG03a0JQZXhobXl5SmpUQ1NDQTZ2VzdNaGRZeUNaZ0p5YitpcHVEOHJ2UmlWbGhBTTBSMS93TUdiM1Nma2UKZXFuc21XUFg1MGF2WXRsMzU1VVIyQUJyeUhiT1c2QkhCTUs4Wm1RRENMOU5zS09qZXFOLzFjdzNkcmlJYksyMgpJb2xLMmduMU8wV3dXME9tc0xQSzlkOExyaXdZNVZMU2FvdHo2MXQvbzVLWk9BTVNCSnJjajVYNkFTeS9YSTBnCjBVYXhRVURWUjFoZlNRUk0zT2dvNUt1ZGZMQlNLN29QMHhRTHNUOEhpbytSeUFYRlZLRmk3RkNnem1wWG9FZEUKTzlhMFBtcGs0NENHCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
        server: https://68C6B19D26ED00FC7DF00376E0FBA706.gr7.eu-west-2.eks.amazonaws.com
      name: kubernetes
    contexts:
    - context:
        cluster: kubernetes
        user: aws
      name: aws
    current-context: aws
    users:
    - name: aws
      user:
        exec:
          apiVersion: client.authentication.k8s.io/v1beta1
          command: /usr/local/bin/aws-iam-authenticator
          args:
            - "token"
            - "-i"
            - "demo-cluster"
    ```

4. **Create AWS Credentials**

5. **Configure Jenkinsfile to Deploy to EKS**

```groovy
pipeline {
    agent any
    stages {
        stage('build app') {
            steps {
               script {
                   echo "building the application..."
               }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                }
            }
        }
        stage('deploy') {
            environment {
                AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
                AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws_secret_access_key')
            }
            steps {
                script {
                   echo 'deploying docker image...'
                   sh 'kubectl create deployment nginx-deployment --image=nginx'
                }
            }
        }
    }
}
```
<img src="https://i.imgur.com/WKwYlT7.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


<img src="https://i.imgur.com/msru2qO.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


<img src="https://i.imgur.com/0U9Ej1S.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>


<img src="https://i.imgur.com/eYDFB0L.png" height="80%" width="80%" alt="Disk Sanitization Steps"/>

