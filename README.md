## 📡 Monitoring F5 Devices with Prometheus and Grafana

This document explains the step-by-step process of integrating F5 devices with Prometheus for monitoring purposes.

---

### 🗺️ Topology Overview

The diagram below shows how F5 devices integrate with Prometheus and Grafana through Telemetry Streaming:

![image](https://github.com/user-attachments/assets/ae4e6208-7157-4cce-99b5-0c7ba4a98ed2)

---

### 🔧 1. Download Telemetry Streaming RPM

Download the required telemetry agent for F5 devices from the link below:

🔗 [https://github.com/F5Networks/f5-telemetry-streaming/releases](https://github.com/F5Networks/f5-telemetry-streaming/releases)

![rpm-download](https://github.com/user-attachments/assets/44d41c5f-aa44-46a1-b6dc-c001cedc7798)

---

### 📦 2. Upload the RPM to F5

In F5 GUI: `iApps > Package Management LX`\
➡️ Click "Import" and upload the downloaded RPM file.

---

### 👤 3. Create a Prometheus User on F5

Navigate to `System > User List > Create`

- **Role:** Administrator
- **Terminal Access:** `tmsh`

![user-defined](https://github.com/user-attachments/assets/c7600f9d-9bac-43ef-96aa-278ebedf7c8c)

---

### 🔐 4. Configure Telemetry via Postman

#### GET (Status Check)

```
URL: https://<F5-MANAGEMENT-IP>/mgmt/shared/telemetry/declare  
Auth: Basic Auth (use the user created in GUI)
```

#### POST (Configuration)

```
URL: https://<F5-MANAGEMENT-IP>/mgmt/shared/telemetry/declare  
Body: raw / JSON  
```

```json
{
  "class": "Telemetry",
  "My_Poller": {
    "class": "Telemetry_System_Poller",
    "interval": 0
  },
  "My_System": {
    "class": "Telemetry_System",
    "enable": "true",
    "systemPoller": ["My_Poller"]
  },
  "metrics": {
    "class": "Telemetry_Pull_Consumer",
    "type": "Prometheus",
    "systemPoller": "My_Poller"
  }
}
```

---

### 📈 5. Verify Metrics via Postman

```
GET
URL: https://<F5-MANAGEMENT-IP>/mgmt/shared/telemetry/pullconsumer/metrics  
BODY: none
```

---

### 🚀 6. Install Prometheus

🔗 [https://prometheus.io/download/](https://prometheus.io/download/)

#### Installation Steps:

```bash
sudo mkdir -p /etc/prometheus
sudo mkdir -p /var/lib/prometheus
cd /tmp
tar -xvzf prometheus-2.52.0.linux-amd64.tar.gz
sudo mv prometheus-2.52.0.linux-amd64 /etc/prometheus
cd /etc/prometheus
vi prometheus.yml
```

#### Example `prometheus.yml`:

```yaml
global:
  scrape_interval: 10s

scrape_configs:
  - job_name: 'TelemetryStreaming'
    scrape_interval: 30s
    scrape_timeout: 30s
    scheme: https
    metrics_path: /mgmt/shared/telemetry/pullconsumer/metrics
    tls_config:
      insecure_skip_verify: true
    basic_auth:
      username: 'prometheus'
      password: '******'
    static_configs:
      - targets: ['<F5-MANAGEMENT-IP>:443']
```

#### Start Prometheus:

```bash
./prometheus
```

---

### 📊 7. Visualize with Grafana

Once Prometheus is running and collecting metrics, configure Grafana to use Prometheus as a data source.
From there, you can create dashboards to visualize F5 performance, traffic, CPU, memory, and more.

---

