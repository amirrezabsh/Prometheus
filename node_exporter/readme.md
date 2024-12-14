Here’s a clean and structured **README** file for setting up Node Exporter on an Ubuntu server.

---

# Node Exporter Setup on Ubuntu Server

This guide explains how to set up **Prometheus Node Exporter** on an Ubuntu server to monitor system metrics such as CPU, memory, and disk usage.

---

## Table of Contents
1. [System Requirements](#system-requirements)  
2. [Download and Install Node Exporter](#download-and-install-node-exporter)  
3. [Create a System User for Node Exporter](#create-a-system-user-for-node-exporter)  
4. [Set Up Node Exporter as a Systemd Service](#set-up-node-exporter-as-a-systemd-service)  
5. [Verify Node Exporter is Running](#verify-node-exporter-is-running)  
6. [Configure Prometheus to Scrape Node Exporter](#configure-prometheus-to-scrape-node-exporter)  
7. [Open Firewall Port (Optional)](#open-firewall-port-optional)  
8. [Visualize Metrics in Grafana](#visualize-metrics-in-grafana)  

---

## 1. System Requirements

- Ubuntu Server 18.04 or newer
- Prometheus server installed and configured
- Root or sudo privileges

---

## 2. Download and Install Node Exporter

1. **Update your system** and download the latest Node Exporter:

   ```bash
   sudo apt update
   wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-1.8.2.linux-amd64.tar.gz
   ```

2. **Extract the downloaded archive**:

   ```bash
   tar xvf node_exporter-1.8.2.linux-amd64.tar.gz
   ```

3. **Move the binary to `/usr/local/bin`**:

   ```bash
   sudo mv node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/
   ```

4. **Clean up unnecessary files**:

   ```bash
   rm -rf node_exporter-1.8.2.linux-amd64.tar.gz node_exporter-1.8.2.linux-amd64
   ```

---

## 3. Create a System User for Node Exporter

To improve security, run Node Exporter as a non-root system user:

```bash
sudo useradd -rs /bin/false node_exporter
```

---

## 4. Set Up Node Exporter as a Systemd Service

1. Create a **systemd service file**:

   ```bash
   sudo nano /etc/systemd/system/node_exporter.service
   ```

2. Add the following configuration:

   ```ini
   [Unit]
   Description=Prometheus Node Exporter
   After=network.target

   [Service]
   User=node_exporter
   ExecStart=/usr/local/bin/node_exporter

   [Install]
   WantedBy=default.target
   ```

3. Save the file and exit (`Ctrl + O`, `Ctrl + X`).

4. Reload `systemd` to recognize the new service:

   ```bash
   sudo systemctl daemon-reload
   ```

5. Start and enable Node Exporter:

   ```bash
   sudo systemctl start node_exporter
   sudo systemctl enable node_exporter
   ```

6. Verify that the service is running:

   ```bash
   sudo systemctl status node_exporter
   ```

---

## 5. Verify Node Exporter is Running

Node Exporter listens on port **9100** by default.

1. Check the metrics using `curl`:

   ```bash
   curl http://localhost:9100/metrics
   ```

2. If successful, you should see a list of metrics.

3. For remote access, replace `localhost` with the server's IP address:

   ```bash
   curl http://<server-ip>:9100/metrics
   ```

---

## 6. Configure Prometheus to Scrape Node Exporter

To allow Prometheus to collect Node Exporter metrics:

1. Add the following configuration to your **Prometheus configuration file** (`prometheus.yml`):

   ```yaml
   scrape_configs:
     - job_name: 'node_exporter'
       static_configs:
         - targets: ['<ubuntu-server-ip>:9100']
   ```

   Replace `<ubuntu-server-ip>` with your server’s IP address.

2. Restart Prometheus to apply changes:

   ```bash
   sudo systemctl restart prometheus
   ```

3. Verify the target is visible in Prometheus at:

   ```
   http://<prometheus-server-ip>:9090/targets
   ```

---

## 7. Open Firewall Port (Optional)

If your server uses **UFW** (Uncomplicated Firewall), open port **9100**:

```bash
sudo ufw allow 9100
```

---

## 8. Visualize Metrics in Grafana

1. Log into your Grafana dashboard.
2. Import a pre-built **Node Exporter** dashboard:
   - Visit [Grafana Dashboards](https://grafana.com/grafana/dashboards).
   - Search for "Node Exporter Full" (e.g., Dashboard ID `1860`).
3. Use Prometheus as the data source to display Node Exporter metrics.

---

## Troubleshooting

- **Service not running**:
  - Check logs: `journalctl -u node_exporter`
- **Metrics not accessible**:
  - Verify port 9100 is open: `sudo ufw status`
- **Prometheus doesn’t scrape data**:
  - Double-check the `prometheus.yml` configuration.

---

## References

- [Prometheus Node Exporter GitHub](https://github.com/prometheus/node_exporter)
- [Prometheus Documentation](https://prometheus.io/docs/)

---

You should now have **Node Exporter** set up and integrated with Prometheus and Grafana for monitoring your Ubuntu server! 🚀
