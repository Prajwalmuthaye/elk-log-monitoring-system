# Centralized Log Monitoring & Alert System using ELK Stack

## Project Overview

This project implements a **centralized log monitoring system** using the ELK Stack (Elasticsearch, Logstash, Kibana) deployed on AWS EC2 with Amazon Linux.

The system collects logs from applications, processes them, stores them centrally, and provides **real-time visualization and alerting**, helping in faster debugging and monitoring.

---

## Features

* Centralized log collection from multiple sources
* Real-time log monitoring
* Log filtering based on severity (INFO, ERROR, DEBUG)
* Interactive dashboards using Kibana
* Alert system for critical issues
* Scalable cloud-based deployment

---

## Architecture

```
Application Logs → Filebeat → Logstash → Elasticsearch → Kibana
                                               ↓
                                            Alerts
```

---

## Tech Stack

| Component              | Purpose                    |
| ---------------------- | -------------------------- |
| Elasticsearch          | Store & search logs        |
| Logstash               | Process and transform logs |
| Kibana                 | Visualization dashboard    |
| Filebeat               | Log collection             |
| Docker                 | Containerization           |
| AWS EC2 (Amazon Linux) | Cloud deployment           |
| Java / Flask           | Log generation             |

---

## Setup & Installation

### Clone Repository

```bash
git clone https://github.com/your-username/elk-log-monitoring-system.git
cd elk-log-monitoring-system
```

---

### 2️Start ELK Stack

```bash
docker-compose up -d
```

---

### Install Filebeat (Amazon Linux)

```bash
sudo yum install filebeat -y
```

---

### Configure Filebeat

Edit:

```bash
sudo nano /etc/filebeat/filebeat.yml
```

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/test.log

output.logstash:
  hosts: ["localhost:5044"]
```

---

### Start Filebeat

```bash
sudo systemctl start filebeat
```

---

## Log Generation (Demo)

```bash
echo "INFO Application Started $(date)" >> /var/log/test.log
echo "ERROR Payment Failed $(date)" >> /var/log/test.log
```

---

## Access Dashboard

Kibana:

```
http://<EC2-IP>:5601
```

Steps:

* Go to **Discover**
* Create index pattern: `logs-*`
* Visualize logs in real-time

---

## Alerting

Alerts can be configured in Kibana:

* Condition: ERROR logs threshold
* Action: Email / webhook notification

---


## Output

* Real-time log monitoring
* Error detection
* Centralized logging system
* Dashboard visualization

---

## Future Scope

* AI-based anomaly detection
* Integration with Kubernetes
* Slack / SMS alerts
* Role-based access control
* Multi-node ELK cluster

---


