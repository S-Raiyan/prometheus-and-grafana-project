# Prometheus + Node Exporter + Grafana Monitoring Setup

## 📌 Project Overview
This project sets up a monitoring stack using **Prometheus**, **Node Exporter**, and **Grafana** to monitor two web servers.  
Prometheus scrapes metrics from Node Exporter, and Grafana visualizes them in dashboards.

---

## ⚙️ Infrastructure
- **Prometheus Server**: `10.0.151.76`
- **Web Server 1**: `10.0.152.44` (Node Exporter)
- **Web Server 2**: `10.0.153.244` (Node Exporter)
- **Grafana**: Installed on Prometheus server (`10.0.151.76`), accessible on port `3000`.

---

## 🚀 Installation Guide

### 1. Install Prometheus
```bash
# Download Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.54.1/prometheus-2.54.1.linux-amd64.tar.gz
tar xvf prometheus-*.tar.gz
cd prometheus-*

# Run Prometheus
./prometheus --config.file=prometheus.yml &


Prometheus runs on port 9090. Access it at:

http://<server-public-ip>:9090


2. Install Node Exporter (on each web server)

wget https://github.com/prometheus/node_exporter/releases/download/v1.8.1/node_exporter-1.8.1.linux-amd64.tar.gz
tar xvf node_exporter-*.tar.gz
cd node_exporter-*
./node_exporter &

Node Exporter runs on port 9100.

3. Configure Prometheus to Scrape Web Servers
Edit prometheus.yml:

scrape_configs:
  - job_name: 'web-servers'
    static_configs:
      - targets: ['10.0.152.44:9100', '10.0.153.244:9100']

Restart Prometheus:
sudo systemctl restart prometheus


4. Install Grafana

# Add Grafana GPG key
wget -q -O - https://packages.grafana.com/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/grafana.gpg

# Add Grafana repository
echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" | \
  sudo tee /etc/apt/sources.list.d/grafana.list

# Update and install Grafana
sudo apt-get update
sudo apt-get install -y grafana

# Enable and start Grafana service
sudo systemctl enable grafana-server
sudo systemctl start grafana-server

Access Grafana at:

http://<server-public-ip>:3000

Default login: admin / admin.

🛠️ Problems Faced & Solutions

❌ Problem 1: Connection timed out when testing Node Exporter
Cause: Security Group (SG) and firewall rules blocked port 9100.

Fix: Added inbound SG rule:

Port: 9100

Source: Prometheus server IP (10.0.151.76/32)

Verified with:

nc -vz 10.0.152.44 9100

❌ Problem 2: Node Exporter reachable on one server but not the other
Cause: Wrong SG/NACL configuration on 10.0.152.44.

Fix: Corrected SG inbound rule and ensured subnet NACL allowed traffic.

After fix:
nc -vz 10.0.152.44 9100
Connection succeeded!


❌ Problem 3: Grafana repo key error (NO_PUBKEY)
Cause: Deprecated apt-key method in Ubuntu.

Fix: Used modern keyring method with signed-by option.

🔗 Connect Prometheus to Grafana
In Grafana → Configuration → Data Sources → Add data source → Select Prometheus.

URL: http://localhost:9090

Save & Test → Success.

📊 Create Dashboard
Imported Node Exporter Full Dashboard (ID: 1860) from Grafana.com.

Dashboard shows:

CPU usage

Memory utilization

Disk usage

Network traffic

System load & uptime

🔥 Stress Test CPU
To verify metrics:

sudo apt-get install -y stress-ng
stress-ng --cpu 4 --timeout 60s


CPU usage spiked in Grafana dashboard, confirming monitoring works.

✅ Final Outcome
Prometheus scrapes metrics from both web servers.

Grafana visualizes metrics in real-time dashboards.

Monitoring stack is fully functional.

Issues with connectivity and repo keys were resolved step by step.

📖 Lessons Learned
Always verify network connectivity with nc or telnet before debugging Prometheus.

Security Groups and NACLs are common causes of scrape failures.

Modern Ubuntu requires signed-by keyring method for external repos.

Stress testing is useful to validate dashboards.

🎯 Next Steps
Add alerting rules in Prometheus (e.g., CPU > 90%).

Configure Grafana alerts.

Extend monitoring to Docker containers and application metrics.

🖼️ Architecture Overview

   +-------------------+
   |   Web Server 1    |
   | Node Exporter 9100|
   +-------------------+
            |
            |
   +-------------------+
   |   Web Server 2    |
   | Node Exporter 9100|
   +-------------------+
            |
            v
   +-------------------+
   |   Prometheus      |
   | Scrapes metrics   |
   | Port 9090         |
   +-------------------+
            |
            v
   +-------------------+
   |     Grafana       |
   | Dashboards 3000   |
   +-------------------+



---

This is the **final, professional README.md file** — ready to copy-paste into your repository. It contains installation guides, troubleshooting, dashboard setup, stress testing, lessons learned, and an architecture diagram all in one place.



