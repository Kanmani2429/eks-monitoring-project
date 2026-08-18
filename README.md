# AWS EKS Monitoring Project

## Overview
A hands-on AWS DevOps project demonstrating how to deploy, monitor, and troubleshoot a Kubernetes application running on Amazon EKS.

The project integrates:

- Prometheus & Grafana for Kubernetes monitoring
- Metrics Server for resource metrics
- Fluent Bit for container log collection
- Amazon CloudWatch for centralized logging
- IAM / IRSA for secure AWS access
- Helm & eksctl for Kubernetes and EKS management

The project focuses on monitoring, centralized logging, AWS IAM integration, and production-style troubleshooting.

## Architecture

                              AWS
                               │
                         Amazon EKS
                              Cluster
                               │
                ┌──────────────┴──────────────┐
                │                             │
                ▼                             ▼
       NGINX Application                   Monitoring Stack
                │                             │
         nginx-service                      Prometheus
                │                             │
                │                           Grafana
                │
                │ Kubernetes Metrics
                ▼
        Metrics Server
                     
             Kubernetes Container Logs
                     │
                     ▼
               Fluent Bit
                     │
                   IRSA
                     │
                     ▼
                 IAM Role
                     │
                     ▼
                IAM Policy
                     │
                     ▼
            Amazon CloudWatch
                     │
                     ▼
                Log Streams 
                

## Technologies

| Category | Technology |
|---|---|
| Cloud | AWS |
| Kubernetes | Amazon EKS |
| Cluster Management | eksctl |
| Package Management | Helm |
| Monitoring | Prometheus |
| Visualization | Grafana |
| Resource Metrics | Metrics Server |
| Logging | Fluent Bit |
| Log Storage | Amazon CloudWatch |
| IAM | IAM / IRSA |
| Infrastructure | CloudFormation |
| CLI | AWS CLI |

## Implementation
1. EKS Cluster

Created an Amazon EKS cluster using eksctl with a managed node group.

The cluster configuration is defined in eks-cluster.yaml.

eksctl create cluster -f eks-cluster.yaml

Validated the cluster:

eksctl get cluster --region us-east-1

aws cloudformation list-stacks \
  --region us-east-1 \
  --stack-status-filter CREATE_IN_PROGRESS CREATE_COMPLETE

eksctl get nodegroup \
  --cluster eks-monitoring-project \
  --region us-east-1

Updated the kubeconfig file:

aws eks update-kubeconfig \
  --name eks-monitoring-project \
  --region us-east-1
  
2. Kubernetes Application

Created a dedicated monitoring namespace and deployed an NGINX application.

kubectl apply -f kubernetes/namespace.yaml

kubectl apply -f kubernetes/deployment.yaml

kubectl apply -f kubernetes/service.yaml

Validated the deployment:

kubectl get pods -n monitoring

kubectl get svc -n monitoring

3. Prometheus and Grafana

Installed the kube-prometheus-stack using Helm.

helm repo add prometheus-community \
  https://prometheus-community.github.io/helm-charts

helm repo update
helm install prometheus \
  prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f helm/kube-prometheus-stack/values.yaml

Prometheus and Grafana were accessed through Kubernetes port forwarding.

4. Kubernetes Metrics

Installed Metrics Server and validated node and pod resource usage.

kubectl apply -f \
  https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

Validated resource metrics:

kubectl top node

kubectl top pods -n monitoring

5. CloudWatch Logging

Created a dedicated CloudWatch log group for Kubernetes application logs.

aws logs create-log-group \
  --log-group-name /eks/eks-monitoring-project/application \
  --region us-east-1

Validated the log group:

aws logs describe-log-groups \
  --log-group-name-prefix /eks/eks-monitoring-project \
  --region us-east-1
  
6. IAM & IRSA Integration

Created an IAM policy allowing Fluent Bit to send logs to CloudWatch.

Configured an IAM OIDC provider for the EKS cluster:

eksctl utils associate-iam-oidc-provider \
  --cluster eks-monitoring-project \
  --region us-east-1 \
  --approve

Created a Kubernetes ServiceAccount connected to an IAM role:

eksctl create iamserviceaccount \
  --cluster eks-monitoring-project \
  --region us-east-1 \
  --namespace amazon-cloudwatch \
  --name fluent-bit \
  --attach-policy-arn arn:aws:iam::<ACCOUNT_ID>:policy/FluentBitCloudWatchPolicy \
  --approve \
  --override-existing-serviceaccounts

This implemented the following authentication flow:

Kubernetes Workload
        │
        ▼
Kubernetes ServiceAccount
        │
        ▼
     IAM Role
        │
        ▼
    IAM Policy
        │
        ▼
Amazon CloudWatch

This avoids storing long-term AWS access keys inside Kubernetes workloads.

7. Fluent Bit

Installed Fluent Bit using Helm to collect Kubernetes container logs.

Added the Fluent Bit Helm repository:

helm repo add fluent \
  https://fluent.github.io/helm-charts

helm repo update

Configured Fluent Bit to use the existing fluent-bit Kubernetes ServiceAccount created and mapped to an IAM role through IRSA.

helm/fluent-bit/values.yaml:

serviceAccount:
  create: false
  name: fluent-bit

Installed Fluent Bit:

helm install fluent-bit fluent/fluent-bit \
  -n amazon-cloudwatch \
  -f helm/fluent-bit/values.yaml

Fluent Bit was configured to:

Read container logs from the Kubernetes nodes
Enrich logs with Kubernetes metadata
Use the fluent-bit ServiceAccount for authentication
Authenticate with AWS using IAM Roles for Service Accounts (IRSA)
Send Kubernetes container logs to Amazon CloudWatch

Validated the Fluent Bit deployment:

kubectl get pods -n amazon-cloudwatch

Checked Fluent Bit logs:

kubectl logs -n amazon-cloudwatch <fluent-bit-pod-name>

Fluent Bit successfully initialized the CloudWatch output plugin and began sending Kubernetes container logs to Amazon CloudWatch.

8. CloudWatch Log Validation

Validated CloudWatch log streams using AWS CLI:

aws logs describe-log-streams \
  --log-group-name /eks/eks-monitoring-project/application \
  --region us-east-1

CloudWatch log streams were successfully created for Kubernetes containers, confirming the complete logging pipeline:

NGINX / Kubernetes Containers
             │
             ▼
        Fluent Bit
             │
             ▼
       IAM / IRSA
             │
             ▼
    CloudWatch Log Group
             │
             ▼
       Log Streams

## Validation
The following components were successfully deployed and validated:

- ✅ Amazon EKS cluster
- ✅ Managed node group
- ✅ Kubernetes namespace
- ✅ NGINX application
- ✅ Kubernetes Service
- ✅ Prometheus
- ✅ Grafana
- ✅ Metrics Server
- ✅ Kubernetes resource metrics
- ✅ Fluent Bit
- ✅ IAM / IRSA integration
- ✅ CloudWatch log group
- ✅ CloudWatch log streams
- ✅ Kubernetes application logs
  
## Key Learnings
	- Amazon EKS cluster and managed node group management
	- Kubernetes application deployment
	- Helm-based application management
	- Prometheus monitoring
	- Grafana visualization
	- Kubernetes Metrics Server
	- Fluent Bit log collection
	- Amazon CloudWatch centralized logging
	- IAM Roles for Service Accounts (IRSA)
	- AWS IAM and OIDC integration
	- Kubernetes troubleshooting
	- Resource and scheduling troubleshooting
	- AWS resource lifecycle management

## Screenshots
Screenshots demonstrating the implementation and validation are included below.

EKS Cluster

<img width="1896" height="232" alt="image" src="https://github.com/user-attachments/assets/13372e84-39b4-4906-abeb-e14dbf567f87" />

<img width="1901" height="112" alt="image" src="https://github.com/user-attachments/assets/e1bf7bdc-5768-43f2-895f-dc07d0b74ed7" />

Kubernetes Application

<img width="1617" height="832" alt="image" src="https://github.com/user-attachments/assets/c3550a6f-a4be-4800-839c-e20b221b7745" />

Prometheus and Grafana

<img width="1116" height="207" alt="image" src="https://github.com/user-attachments/assets/81fe7817-6d08-4375-bf50-5bc8237f12be" />

<img width="1775" height="545" alt="image" src="https://github.com/user-attachments/assets/b8411b3f-b81f-4c2e-9a9d-b9d1091ea116" />

<img width="885" height="937" alt="image" src="https://github.com/user-attachments/assets/75588d12-036c-4e40-939d-2671dd4a68bc" />

Kubernetes Metrics

<img width="1430" height="506" alt="image" src="https://github.com/user-attachments/assets/620e3932-f657-4455-896c-c81ff6ff6cab" />

Fluent Bit

<img width="1167" height="72" alt="image" src="https://github.com/user-attachments/assets/e249f00b-01f1-4c67-8c1e-453fb119e82b" />

CloudWatch Logs

<img width="1881" height="732" alt="image" src="https://github.com/user-attachments/assets/81052999-d34b-40c3-93f4-b59ff334f851" />

# Project Status
Completed
The project successfully demonstrated an end-to-end Kubernetes monitoring and logging workflow on Amazon EKS, including monitoring, metrics collection, centralized logging, IAM integration, validation and troubleshooting.




























