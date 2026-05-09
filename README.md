# Centralized Log Monitoring & Alert System using ELK Stack

## Project Overview
This project implements a centralized log monitoring and alerting system using the ELK Stack. Logs generated from a remote website server are collected using Filebeat and sent to a centralized Logstash server. Logstash processes the logs and forwards them to Elasticsearch. Kibana is used for visualization, dashboard creation, and alert monitoring.

## Technologies Used
- AWS EC2
- Docker
- Elasticsearch 8.12.0
- Logstash 8.12.0
- Kibana 8.12.0
- Filebeat
- Apache Tomcat
- Java J2EE Web Application
- Linux / Amazon Linux 2023

## System Architecture

```text
Website EC2 Instance
Online Book Store Web App
Tomcat Docker Container
Application Logs
Filebeat
        |
        | Port 5044
        v
ELK EC2 Instance
Logstash
Elasticsearch
Kibana Dashboard
Alert System
```
Project Features
Centralized log collection
Remote server log monitoring
Real-time log visualization
Error log detection
Kibana dashboard
Alert generation
AWS cloud deployment
Docker-based ELK setup
EC2 Instances Used
1. ELK Server

This server runs:

Elasticsearch
Logstash
Kibana
2. Website Server

This server runs:

OnlineBookStore web application
Tomcat Docker container
Filebeat agent

# ELK Server Setup
Step 1: Install Docker
```
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
```
Step 2: Create Project Directory
```
mkdir elk-project
cd elk-project
```
Step 3: Create docker-compose.yml
```
nano docker-compose.yml
```
Paste:
```
version: '3'

services:

  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.12.0
    container_name: elasticsearch
    environment:
      discovery.type: single-node
      xpack.security.enabled: "false"
      ES_JAVA_OPTS: "-Xms256m -Xmx256m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"

  logstash:
    image: docker.elastic.co/logstash/logstash:8.12.0
    container_name: logstash
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.12.0
    container_name: kibana
    environment:
      ELASTICSEARCH_HOSTS: "http://elasticsearch:9200"
      XPACK_SECURITY_ENABLED: "false"
      XPACK_ENCRYPTEDSAVEDOBJECTS_ENCRYPTIONKEY: "elkstackprojectencryptionkey2026secure"
      XPACK_SECURITY_ENCRYPTIONKEY: "elkstacksecurityencryptionkey2026secure"
      XPACK_REPORTING_ENCRYPTIONKEY: "elkstackreportingencryptionkey2026secure"
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
```
Step 4: Create Logstash Configuration
```
nano logstash.conf
```
Paste:
```
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => { "message" => "%{GREEDYDATA:log}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://elasticsearch:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }

  stdout {
    codec => rubydebug
  }
}
```
Step 5: Start ELK Stack
```
docker-compose up -d
```
Step 6: Verify Containers
```
docker ps
```
Expected containers:

elasticsearch
logstash
kibana
Step 7: Check Elasticsearch
```
curl http://localhost:9200
```
Step 8: Check Cluster Health
```
curl http://localhost:9200/_cluster/health?pretty
```
Security Group Configuration
ELK Server Inbound Rules
Port	Purpose
22	SSH
5044	Filebeat to Logstash
5601	Kibana
9200	Elasticsearch testing

Allow port 5044 from the Website EC2 private IP.

# Website Server Setup
Step 1: Install Docker
```
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
```
Step 2: Build Java Web Application
```
cd onlinebookstore-J2EE
mvn clean package
```
cd onlinebookstore-J2EE
mvn clean package

Step 3: Run Tomcat Container
```
docker run -d \
  --name onlinebookstore \
  -p 8080:8080 \
  -v $(pwd)/target/onlinebookstore-0.0.1-SNAPSHOT.war:/usr/local/tomcat/webapps/onlinebookstore.war \
  tomcat:9.0
```
Step 4: Verify Web Application
```
docker ps
docker logs onlinebookstore
```
Open in browser:
```
http://WEBSITE_EC2_PUBLIC_IP:8080/onlinebookstore
```
Application Log File Setup

Create a log file from Tomcat container logs:
```
docker logs -f onlinebookstore > /var/log/onlinebookstore.log 2>&1 &
```
Generate test log:
```
echo "ERROR OnlineBookStore Test $(date)" | sudo tee -a /var/log/onlinebookstore.log
```
# Filebeat Setup on Website Server
Step 1: Add Elastic Repository
```
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
```
cat <<EOF | sudo tee /etc/yum.repos.d/elastic.repo
[elastic-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
```
Step 2: Install Filebeat
```
sudo yum install filebeat -y
```
Step 3: Configure Filebeat
```
sudo nano /etc/filebeat/filebeat.yml
```
Use this configuration:
```
filebeat.inputs:
- type: filestream
  enabled: true
  paths:
    - /var/log/onlinebookstore.log

output.logstash:
  hosts: ["ELK_SERVER_PRIVATE_IP:5044"]
```
Step 4: Restart Filebeat
```
sudo systemctl restart filebeat
sudo systemctl enable filebeat
```
Step 5: Test Filebeat Output
```
sudo filebeat test output
```
sudo filebeat test output
Expected result:

logstash: 192.168.1.164:5044
connection... OK

# Verify Logs in Elasticsearch

On ELK server:
```
curl http://localhost:9200/_cat/indices?v
```
# Kibana Setup

Open Kibana:
```
http://ELK_SERVER_PUBLIC_IP:5601
```
Create Data View

Go to:
```
Stack Management → Data Views → Create Data View
```
Use:
```
logs-*
```
Timestamp field:
```
@timestamp
```
Save data view.
# Kibana Discover

Go to:
```
Discover
```
Dashboard Ideas

Create dashboard:
```

Centralized Log Monitoring Dashboard
```
# Kibana Alert Creation

Go to:
```
Stack Management → Rules and Connectors → Create Rule
```
Select:

Log threshold

Configuration:
```
Name: High Error Log Alert
Log View: Default
Condition: message matches phrase ERROR
Threshold: more than 0
Time Window: last 30 minutes
```
Action:
```
Connector: Server Log
Level: Error
```
Message:
```
Alert: High ERROR logs detected.

Rule Name: {{rule.name}}
Reason: {{context.reason}}

Matched Logs: {{context.matchingDocuments}}
Condition: {{context.conditions}}

Please check Kibana dashboard for detailed log analysis.
```
# Testing Alert

On Website server:
```
echo "ERROR Payment Failed $(date)" | sudo tee -a /var/log/onlinebookstore.log
```
## Project Workflow
```
User opens website
        ↓
Tomcat generates logs
        ↓
Logs are written to /var/log/onlinebookstore.log
        ↓
Filebeat reads log file
        ↓
Filebeat sends logs to Logstash on port 5044
        ↓
Logstash processes logs
        ↓
Elasticsearch stores logs
        ↓
Kibana visualizes logs
        ↓
Alert triggers on ERROR logs
```
