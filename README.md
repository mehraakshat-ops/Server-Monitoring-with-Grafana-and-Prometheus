# Server-Monitoring-with-Grafana-and-Prometheus

## Table of Contents
- [Description](#description)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Installation Steps](#installation-steps)
- [Usage](#usage)

## Description
This project demonstrates how to set up Prometheus and Grafana using Docker to monitor system metrics on a target server. By following this setup, users can collect and visualize server data in real-time, such as CPU, memory, and network usage, enabling effective monitoring and troubleshooting.

The process involves:

- Setting Up Prometheus: We configure Prometheus on a dedicated monitoring server, setting the scrape intervals and target IPs to periodically collect metrics from the target server.
- Setting Up Node Exporter: We configure the target server as a monitoring endpoint using Node Exporter, which exposes system metrics (CPU, memory, disk, etc.) that Prometheus collects.
- Integrating with Grafana: Grafana is configured to use Prometheus as its data source and visualize metrics through dashboards, allowing detailed analysis of server health and performance.
  
Technologies used:

- Prometheus: For metric collection and monitoring.
- Node Exporter: To expose server metrics for Prometheus.
- Grafana: For data visualization with customizable dashboards.
- Docker: For containerizing each component for ease of deployment and scalability.
This project is designed for users looking to monitor their server infrastructure effectively, especially useful in environments where proactive performance tracking is critical. The setup is modular and can be adapted to different server configurations, making it versatile for various monitoring needs.


## Features

1. **Containerized Setup**:
   - Prometheus, Node Exporter, and Grafana are containerized with Docker, ensuring easy setup, portability, and resource isolation across servers.

2. **Real-Time Server Monitoring**:
   - Prometheus collects real-time system metrics (CPU, memory, disk usage, network traffic) from the target server via Node Exporter, with configurable scrape intervals for optimized performance.

3. **Detailed Metrics Visualization**:
   - Grafana provides an interactive, customizable dashboard to visualize the collected data from Prometheus. This includes CPU utilization, memory consumption, disk I/O, and network traffic, with easy filtering options.

4. **Configurable Data Source**:
   - Grafana’s data source configuration allows for seamless integration with Prometheus and provides flexibility to add more data sources if needed.

5. **High Customizability**:
   - Prometheus scrape intervals, target IPs, and Grafana dashboard configurations are easily customizable, enabling users to tailor the monitoring to specific requirements.

6. **Role-Based Targeting**:
   - With Node Exporter deployed on target servers, this setup can be expanded to monitor multiple servers by updating the Prometheus configuration with new target IPs.

7. **Firewall Configuration for Security**:
   - The project includes firewall configurations, ensuring that only required ports are open for Prometheus to access metrics on target servers securely.

8. **Open-Source Dashboard Templates**:
   - Uses Grafana’s Node Exporter Full Dashboard template (ID 1860) from Grafana Labs, providing comprehensive, pre-configured visualizations for common system metrics.

9. **Data Persistence**:
   - Docker volumes are configured to persist Prometheus and Grafana data across container restarts, ensuring continuity in metric collection and visualization.
  

## Tech Stack

1. **Prometheus**:
   - **Role**: Metrics collection and monitoring.
   - **Functionality**: Collects and stores real-time system metrics from target servers, with customizable scrape intervals for efficiency.

2. **Node Exporter**:
   - **Role**: Metric exporter for Prometheus.
   - **Functionality**: Installed on target servers to expose system metrics (CPU, memory, disk, network) for Prometheus to collect.

3. **Grafana**:
   - **Role**: Data visualization and dashboarding.
   - **Functionality**: Connects to Prometheus as a data source to visualize metrics, enabling customizable dashboards for monitoring server performance.

4. **Docker**:
   - **Role**: Containerization.
   - **Functionality**: Hosts Prometheus, Node Exporter, and Grafana in isolated containers, ensuring consistency and ease of deployment.

5. **Docker Compose**:
   - **Role**: Multi-container orchestration.
   - **Functionality**: Defines and runs multiple Docker containers (Prometheus, Node Exporter, Grafana) in a single configuration file, streamlining setup and management.

6. **Linux Firewall**:
   - **Role**: Security.
   - **Functionality**: Configures port access (e.g., 9100 for Node Exporter) to allow secure metric collection between Prometheus and target servers.

7. **Rocky Linux 9**:
   - **Role**: Operating System.
   - **Functionality**: Provides a stable, secure base for Docker and server applications in this setup.

This stack provides a scalable, secure, and easily deployable solution for monitoring infrastructure health and performance in real-time.


## Installation Steps

### Prerequisites

- **Linux Servers**: Ensure you have two Linux servers (e.g., Rocky Linux 9), one for Prometheus and Grafana, and the other as a target server for monitoring.
- **Docker and Docker Compose**: Install Docker and Docker Compose on both servers.

### 1. Set Up Prometheus on Monitoring Server

1. **SSH into Monitoring Server**:
   ```bash
   ssh user@monitoring-server-ip
   ```

2. **Create Required Directories**:
   ```bash
   sudo mkdir -p /opt/docker/prometheus
   ```

3. **Navigate to Prometheus Directory**:
   ```bash
   cd /opt/docker/prometheus
   ```

4. **Create Docker Compose File**:
   Create a `docker-compose.yml` file with the following content:

   ```yaml
   version: '3'
   services:
     prometheus:
       image: prom/prometheus:latest
       container_name: prometheus
       ports:
         - "9090:9090"
       volumes:
         - ./prometheus:/etc/prometheus
         - ./prometheus-data:/prometheus
       command:
         - '--config.file=/etc/prometheus/prometheus.yml'
       restart: unless-stopped
   ```

5. **Create Prometheus Configuration File**:
   In the `/opt/docker/prometheus/prometheus` directory, create a file named `prometheus.yml`:

   ```yaml
   global:
     scrape_interval: 15s
   scrape_configs:
     - job_name: 'node_exporter'
       scrape_interval: 5s
       static_configs:
         - targets: ['target-server-ip:9100']
   ```

6. **Run Prometheus**:
   ```bash
   docker-compose up -d
   ```

### 2. Set Up Node Exporter on Target Server

1. **SSH into Target Server**:
   ```bash
   ssh user@target-server-ip
   ```

2. **Create Required Directories**:
   ```bash
   sudo mkdir -p /opt/docker/node_exporter
   ```

3. **Navigate to Node Exporter Directory**:
   ```bash
   cd /opt/docker/node_exporter
   ```

4. **Create Docker Compose File**:
   Create a `docker-compose.yml` file with the following content:

   ```yaml
   version: '3'
   services:
     node_exporter:
       image: prom/node-exporter:latest
       container_name: node_exporter
       network_mode: host
       restart: unless-stopped
   ```

5. **Open Firewall Port for Node Exporter**:
   ```bash
   sudo firewall-cmd --zone=public --add-port=9100/tcp --permanent
   sudo firewall-cmd --reload
   ```

6. **Run Node Exporter**:
   ```bash
   docker-compose up -d
   ```

### 3. Set Up Grafana on Monitoring Server

1. **Navigate to Grafana Directory on Monitoring Server**:
   ```bash
   cd /opt/docker && sudo mkdir grafana && cd grafana
   ```

2. **Create Docker Compose File**:
   Create a `docker-compose.yml` file with the following content:

   ```yaml
   version: '3'
   services:
     grafana:
       image: grafana/grafana:latest
       container_name: grafana
       ports:
         - "3000:3000"
       volumes:
         - ./grafana-data:/var/lib/grafana
       restart: unless-stopped
   ```

3. **Run Grafana**:
   ```bash
   docker-compose up -d
   ```

### 4. Configure Grafana

1. **Access Grafana**: Open a browser and navigate to `http://monitoring-server-ip:3000`.
2. **Login**: The default credentials are `admin` / `admin`. You will be prompted to change the password.
3. **Add Prometheus as a Data Source**:
   - Go to **Settings** > **Data Sources** > **Add Data Source**.
   - Choose **Prometheus** and enter the Prometheus URL (`http://monitoring-server-ip:9090`).
   - Click **Save & Test** to confirm connectivity.
4. **Import Node Exporter Dashboard**:
   - Go to **Create** > **Import**.
   - Enter the Grafana dashboard ID `1860` for the Node Exporter Full dashboard and select Prometheus as the data source.


With this setup, you should now be able to monitor system metrics on the Grafana dashboard using Prometheus data from Node Exporter on the target server.


## Usage

### 1. Access Prometheus

Prometheus is now set up to scrape metrics from the Node Exporter on the target server.

- **Prometheus Web UI**: Open your browser and navigate to:
  ```
  http://<monitoring-server-ip>:9090
  ```
- **Querying Metrics**:
  - In the Prometheus Web UI, you can enter metric names to view specific data. For example:
    - `node_cpu_seconds_total` for CPU usage.
    - `node_memory_MemAvailable_bytes` for available memory.
    - `node_network_receive_bytes_total` and `node_network_transmit_bytes_total` for network usage.

### 2. Access Grafana

Grafana provides an interactive dashboard for visualizing metrics.

- **Grafana Web UI**: Open your browser and go to:
  ```
  http://<monitoring-server-ip>:3000
  ```
- **Login Credentials**: Use the credentials set up during installation. The default is:
  - **Username**: `admin`
  - **Password**: `admin` (you may have changed this at first login)

### 3. Viewing System Metrics on Grafana

1. **Open the Dashboard**:
   - After logging in to Grafana, navigate to the dashboard by selecting **Dashboards** > **Manage** and choosing the Node Exporter Full dashboard (ID `1860`).
   
2. **Visualize Metrics**:
   - The dashboard will display various system metrics, such as CPU usage, memory utilization, disk I/O, and network statistics, in real-time.
   - You can hover over individual graphs for detailed insights or customize the view based on your monitoring needs.

3. **Adjust Time Range**:
   - Use the time picker in the top-right corner of the Grafana UI to set the desired time range (e.g., last 5 minutes, last 1 hour, etc.).

### 4. Customizing Prometheus Alerts (Optional)

To receive alerts based on specific conditions, you can configure alert rules in Prometheus.

1. **Edit Prometheus Configuration**:
   - Open the `prometheus.yml` file in the Prometheus configuration directory (`/opt/docker/prometheus/prometheus`).
   
2. **Define Alerting Rules**:
   - Add custom alert rules based on metrics, such as CPU usage thresholds, memory usage, or disk space.

3. **Reload Prometheus Configuration**:
   - After editing the configuration, restart the Prometheus service to apply changes:
     ```bash
     cd /opt/docker/prometheus
     docker-compose restart prometheus
     ```

With this setup, your monitoring system is fully functional and providing insights into your target server’s performance and health through Grafana dashboards and Prometheus metrics.



## Project Structure

```
.
├── docker-compose.yml               # Main Docker Compose file to start Prometheus, Node Exporter, and Grafana containers
├── prometheus/
│   ├── docker-compose.yml           # Docker Compose configuration for Prometheus
│   ├── prometheus.yml               # Prometheus configuration file (scrape intervals, target servers, etc.)
│   ├── prometheus/                  # Directory for Prometheus container volume
│   └── prometheus-data/             # Directory for Prometheus data storage
├── node_exporter/
│   ├── docker-compose.yml           # Docker Compose configuration for Node Exporter on target server
│   └── node_exporter/               # Directory for Node Exporter container volume
└── grafana/
    ├── docker-compose.yml           # Docker Compose configuration for Grafana
    ├── grafana/                     # Directory for Grafana container volume
    └── grafana-data/                # Directory for Grafana data storage
```

### Folder Descriptions

- **prometheus/**: Contains all Prometheus-related configurations and data.
  - `prometheus.yml` - Configuration file for Prometheus with scrape intervals and target servers.
  - `prometheus-data/` - Persistent storage for Prometheus data.
- **node_exporter/**: Contains configuration for Node Exporter on the target server to export server metrics.
- **grafana/**: Contains all Grafana-related configurations and data.
  - `grafana-data/` - Persistent storage for Grafana dashboards and settings.

Each component (Prometheus, Node Exporter, Grafana) is structured in its own directory for easier maintenance and organization. Adjust file paths as necessary for your setup.

---



  
