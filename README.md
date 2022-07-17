# CICD Pipeline

### Stages of CICD Pipeline For Build and Deploy
1. Clone: checks out the source code to jenkins workspace.
2. Build & Test: Installs the dependencies and run the unit test.
3. Build Docker image: Docker Container image is created.
4. Push Image to Hub: Docker image is now pushed to Dockerhub.
5. Deploy to K8s: Deployment.yaml file deploys image to the K8s using by validdating kubeconfig.

### High lights of my CICD pipeline

#### **Jenkins pipeline view**
![Jenkins pipeline view](images/CICDpipelinedeploytoK8scluster1.png "Title")
### References
* **Sample App for** demo:  
![Sample App for demo](images/service-exposed.png "Title")
 * Pipeline Implimentation:https://github.com/doddabasappa94/devops-automation

### Stages of CICD Pipeline for Parallel Build and Deploy
1. Checkout: checks out the source code to jenkins workspace.
2. Build & Test: Installs the dependencies and run the unit test.
3. Build Docker image: Both Docker Container image is created.
4. Push Image to Hub: Both Docker image is now pushed to Dockerhub.
5. Deploy to K8s: Both Deployment.yaml file deploys image to the K8s Cluster.

### High lights of my CICD pipeline Parallel Build and Deploy

#### **Jenkins pipeline view Parallel Build and Deploy**
![Jenkins pipeline view](images/Parallel-build-and-deploy.png "Title")
#### **CICD Process**
Developer pushes the source code to his git branch and immediately triggers the jenkins job. Jenkins builds, tests and containerizes the app and at last it will deploy it to K8s cluster. This is the path path, This is happens if all criterias are met .

### Refernces
* **Sample App for** demo:  
![Sample App for demo](images/service-exposed.png "Title")
* pipeline implementation: https://github.com/doddabasappa94/parallel-devops


# Infrastructure as code 

**Following Modules are used to create Infrastructure**
* vpc 
* Ec2  
* Rds

**Note**
* These modules are tailored to suite my requirement

## Terraform template strucure
```
├── module  #--------------------------> module definition
│   ├── Ec2
│   │   ├── instance.tf
│   │   ├── variable.tf
│   │   ├── securityGroup.tf
│   │   └── variable.tf
│   ├── Rds
│   │   ├── mysql.tf
│   │   ├── securityGroup.tf
│   │   └── variable.tf
│   └── vpc
│       └── vpc.tf
│       ├── public_subnet.tf
│       ├── private_subnet.tf 
|       ├── route_table.tf
│       ├── internet_gateway.tf
│       ├── nat.tf 
│       ├── variable.tf

├── main.tf      #------------------------> module declaration
├── provider.tf  #------------------------> Provider definition
├── backend.tf   #------------------------> Backend configuration
├── variables.tf #------------------------> Variable declaration
└── Readme.md    #------------------------> Command Reference for terraform script execution 
```

### Salient features of the terraform template
1. Template Creates following things using vpc module.
    - vpc
    - 3 Public subnets and 3 Private subnets
    - Route table & Route association
    - internet and Nat Gateways
2. Template Creates following things using Ec2 module.
    - 3 EC2 instances created
    - security group created
3. Template Creates following things using Rds module.
    - Rds instance created  
4. Template is workspaced, same template can be used to bootstrap multiple cluster using multiple workspaces.

# Application Monitoring 
* Cluster and application monitoring is crucial for any organization whose applications run on clusters. Any problem with the cluster can lead to a huge loss to the organization. 
* For current implementation, Prometheus Opertor and associated tools like kube-state metrics, node-exporter , alert manager are used to monitor the cluster components.
* Applcations related metrics can also be monitored through prometheus if the instrumentation is implemented at the microservice level.

### About Prometheus Operator
* Prometheus Operator uses CRD (Custom Resource Definitions) to generate configuration files and identify Prometheus resources.
    * alertmanagers – defines installation for Alertmanager
    * podmonitors – determines which pods should be monitored
    * prometheuses – defines installation for Prometheus
    * prometheusrules – defines rules for alertmanager
    * servicemonitors – determines which services should be monitored
* The operator monitors Prometheus resources and generates Deployments (Prometheus and Alertmanager) and configuration files (prometheus.yaml, alertmanager.yaml).

* Current operator chart deploys following components 
    * Prometheus Operator
    * Prometheus
    * Alertmanager
    * Prometheus node-exporter
    * kube-state-metrics
    * Grafana

#### **Prometheus Architecture**
![Prometheus Architecture](images/prom-arch.png "Title")
#### **Prometheus Components**
![Prometheus Components](images/Promethues-grafana-pods.png "Title")
#### **Prometheus Operator Workflow**
![Prometheus Operator Architecture](images/prometheus-op-architecture.png "Title")


#### **Features**
1. All configuration are stored in declarative manner.
### Cluster Monitoring 
Cluster is monitored using the Prometheus stack. Deployment is done through helm chart 

#### **Cluster Metrics**
![Cluster Metrics](images/Clustermetrics.png "Title")
### Pod Application Monitoring 
#### ** Pod Application Metrics**

![Pod Metrics](images/Podmetrics.png "Title")
## **Prometheus Alert manager**: 
* Service monitors that describe and manage alerts to the targets like email and slack to be scraped by Prometheus. The Prometheues describe Rules to trigger a alert based Conditions..

### Exmaple implementation of service monitor for application

* **Alert Manager ConfigMap for app**
```
kind: ConfigMap
apiVersion: v1
metadata:
  name: alertmanager-config
  namespace: monitoring
data:
  config.yml: |-
    global:
    templates:
    - '/etc/alertmanager/*.tmpl'
    route:
      receiver: alert-emailer
      group_by: ['alertname', 'priority']
      group_wait: 10s
      repeat_interval: 30m
      routes:
        - receiver: prometheusalert
        # Send severity=slack alerts to slack.
          match:
            severity: slack
          group_wait: 10s
          repeat_interval: 1m
 
    receivers:
    - name: alert-emailer
      email_configs:
      - to: abc1234@gmail.com
        send_resolved: false
        from: slackalerts24@gmail.com
        smarthost: smtp.gmail.com:587
        auth_username: "slackalerts24@gmail.com"
        auth_identity: "slackalerts24@gmail.com"
        auth_password: "abc@123"
        require_tls: false
    - name: prometheusalert
      slack_configs:
      - api_url: https://hooks.slack.com/services/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
        channel: '#prometheusalert'

```
#### Prometheus Alert Manager Notifications In Slack
![Slack Alerts](images/slackalert.png "Title") 

# ELK Stack Log Monitoring(Elasticsearch Stack: Elasticsearch, Logstash, Kibana)
* ELK stack is a combination of three open-source tools that form a log management platform that specializes in searching, analyzing, and visualizing logs generated from kubernets pods.
* Prometheus is mainly we used for metrics while Elk Stack is mainly to aggregate logs from all your applications, analyze these logs, and create visualizations for application.
* Elastic Search, Kibana, and Logstash make up the ELK stack. Log aggregation is the major goal of this. The development of micro-service architecture needs a better method for collecting and searching logs for debugging purposes. These logs can be gathered and explored using the ELK stack.

## ELK Stack Components:
* **Elastic Search:** This is the database which stores all the logs,he indexer and search engine used to store the data gathered by Beats and Logstash.
* **Kibana:** an interface to Elasticsearch providing many types of visualization to help analyse and understand the data.
* **Logstash:** collects and processes data in many formats, allowing parsing and enrichment.
* **Filebeat:** Filebeat is very important component and works as the log exporter. It exports and forwards the log to Logstash.

### ELK stack Architecture
![Elk Architecture](images/ELK.png "Title") 

### ELK stack Application K8s Pods

![Elk Pods](images/elkstackpodssvc.png "Title") 

### Application logs

![ Application logs](images/elasticsearchK8slogs.png "Title") 


# Shell Scripting
### Shellscript identify IP Regex and replace it with 127.0.0.1
###### Example file ipfile.txt
```
192.53.65.32 12.3.33.98.56
198.26.36.26 13.2332.6565.56
192.22.23.63 25.36.12.3
10.26.16.12 8.8.8.8
0.0.0.0 1.1.1.1
```
#### Ipregex.sh
```
#!/bin/bash

#replace all ip with 127.0.0.1
sed -r 's/[0-9]{1,3}[.][0-9]{1,3}[.][0-9]{1,3}[.][0-9]{1,3}/127.0.0.1/g' $1
```
### For the Below CSV write a Shellscript to update 'C' Salary to 3000
###### csv file abc.csv
```
Name,Age,Gender,Salary
A,12,M,3000
B,12,M,2000
C,12,M,1000
```
#### csalary.sh
```
#! /bin/bash
while IFS="," read -r rec_column1 rec_column2 rec_column3 rec_column4
do
	if [ $rec_column1 == "C" ];then
          sed -i -r "s/${rec_column4}/3000/g" abc.csv
          echo "Displaying Record-$rec_column1"
          echo "Age: $rec_column2"
          echo "Gender: $rec_column3"
          echo "Salary: $rec_column4"
	fi
done < <(tail -n +2 abc.csv)
```
 ##### or
 ```
 awk 'BEGIN{FS=OFS=","} $1=="C"{$4="3000"} 1' abc.csv
 ```
### Delete Crashlooping Pods in Kubernetes Cluster
#### Crashloop.sh
```
#!/bin/bash
kubectl get pods --all-namespaces | grep 'CrashLoopBackOff' | awk '{print $2 " --namespace=" $1}' | xargs kubectl delete pod echo "$2 is deleted"
```
