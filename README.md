# CICD Pipeline

### Stages of CICD Pipeline 1
1. Clone: checks out the source code to jenkins workspace.
2. Build & Test: Installs the dependencies and run the unit test.
3. Build Docker image: Docker Container image is created.
4. Push Image to Hub: Docker image is now pushed to Dockerhub.
5. Deploy to K8s: Deployment.yaml file deploys image to the K8s using by validdating kubeconfig.

### High lights of my CICD pipeline

#### **Jenkins pipeline view**
![Jenkins pipeline view](images/CICDpipelinedeploytoK8scluster1.png "Title")

### Stages of CICD Pipeline for Parallel build and deploy
1. Checkout: checks out the source code to jenkins workspace.
2. Build & Test: Installs the dependencies and run the unit test.
3. Build Docker image: Both Docker Container image is created.
4. Push Image to Hub: Both Docker image is now pushed to Dockerhub.
5. Deploy to K8s: Both Deployment.yaml file deploys image to the K8s Cluster.

### High lights of my CICD pipeline

#### **Jenkins pipeline view**
![Jenkins pipeline view](images/Parallel-build-and-deploy.png "Title")
#### **CICD Process**
Developer pushes the source code to his git branch and immediately triggers the jenkins job. Jenkins builds, tests and containerizes the app and at last it will deploy it to K8s cluster. This is the path path, This is happens if all criterias are met .

### Refernces
* **Sample App for** demo:  
![Sample App for demo](images/service-exposed.png "Title")
* pipeline implementation: https://github.com/doddabasappa94/parallel-devops


# Infrastructure as code 

**Following Modules are used to create eks Cluster**
* vpc 
* eks cluster
* managed_node_group  

**Note**
* These modules are tailored to suite my requirement

## Terraform template strucure
```
├── module  #--------------------------> module definition
│   ├── Ec2
│   │   ├── .tf
│   │   ├── iam.tf
│   │   ├── locals.tf
│   │   ├── securityGroup.tf
│   │   └── variable.tf
│   ├── node_group
│   │   ├── iam.tf
│   │   ├── locals.tf
│   │   ├── nodeGroup.tf
│   │   ├── output.tf
│   │   └── variable.tf
│   └── vpc
│       ├── gateways.tf
│       ├── locals.tf
│       ├── output.tf
│       ├── route_table.tf
│       ├── subnets.tf
│       ├── variable.tf
│       └── vpc.tf
├── main.tf      #------------------------> module declaration
├── provider.tf  #------------------------> Provider definition
├── backend.tf   #------------------------> Backend configuration
├── variables.tf #------------------------> Variable declaration
└── Readme.md    #------------------------> Command Reference for terraform script execution 
```

### Salient features of the terraform template
1. Template Creates following things using vpc module.
    - vpc
    - subnet
    - Route table & Route association
    - internet and Nat Gateways
2. Template Creates following things using eks module.
    - iam role required by eks
    - security group for eks
    - eks cluster 
3. Template Creates following things using node_group module.
    - iam role used by eks worker node group
    - security group used by eks worker node group
    - private managed node group     
    - public managed node group    
4. Terraform saves the state in s3 bucket. Terraform template is executed from the jenkins job. Concurrent builds are disabled, so state file locking is not needed. 

5. Template is workspaced, same template can be used to bootstrap multiple cluster using multiple workspaces.

6. Provisioning is done using jenkins pipeline.

#### **Infra Pipeline Stages**
![Infra Pipeline Stages](img/Infra_pipeline.png "Title")
### Refernces
* Terraform template: https://github.com/navaganeshr/eks_cluster
* pipeline implementation: https://github.com/navaganeshr/infra-pipelines

# Application Monitoring 
* Cluster and application monitoring is crucial for any organization whose applications run on clusters. Any problem with the cluster can lead to a huge loss to the organization. 
* For current implementation, Prometheus Opertor and associated tools like kube-state metrics, nodeexporter , blackbox exporter are used to monitor the eks cluster components.
* Applcations related metrics can also be monitored through prometheus if the instrumentation is implemented at the microservice level.

### About Prometheus Operator
* Prometheus Operator uses CRD (Custom Resource Definitions) to generate configuration files and identify Prometheus resources.
    * alertmanagers – defines installation for Alertmanager
    * podmonitors – determines which pods should be monitored
    * prometheuses – defines installation for Prometheus
    * prometheusrules – defines rules for alertmanager
    * servicemonitors – determines which services should be monitored
* The operator monitors Prometheus resources and generates StatefullSet (Prometheus and Alertmanager) and configuration files (prometheus.yaml, alertmanager.yaml).

* Current operator chart deploys following components 
    * Prometheus Operator
    * Prometheus
    * Alertmanager
    * Prometheus node-exporter
    * kube-state-metrics
    * Grafana

#### **Prometheus Architecture**
![Prometheus Components](img/prom-arch.jpg "Title")
#### **Prometheus Components**
![Prometheus Components](img/prometheus_components.png "Title")
#### **Prometheus Operator Workflow**
![Prometheus Operator Architecture](img/prometheus-op-architecture.png "Title")


#### **Features**
1. All configuration are stored in declarative manner.
### Cluster Monitoring 
Cluster is monitored using the Prometheus stack. Deployment is done through helm chart 


#### **Cluster Metrics**
![Cluster Metrics](img/ClusterMetrics.png "Title")
### Application Monitoring 
#### **Application Metrics**

![Cluster Metrics](img/app-metrics.png "Title")
## **service monitors**: 
* Service monitors that describe and manage monitoring targets to be scraped by Prometheus. The Prometheus resource connects to ServiceMonitors using a **serviceMonitorSelector** field. This way Prometheus sees what targets (apps) have to be scraped.

### Exmaple implementation of service monitor for application

* **Service Monitor Definition for app**
```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nodejs-hello-world
  labels:
    env: dev
    release: monitoring-stack
spec:
  selector:
    matchLabels:
      app: rpc-app
  endpoints:
  - port: web
```

* **Prometheus config for selecting targets**

```
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
  metadata:
    name: prometheus
spec:
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      release: monitoring-stack
  resources:
    requests:
      memory: 400Mi
```
## How is Prometheus opertor is Deployed ?
- Prom Opertor helm chart is deployed though jenkins. 
references: 
  - Jenkins pipeline: https://github.com/navaganeshr/platform-pipelines
  - helm chart : https://github.com/navaganeshr/platform-tools
