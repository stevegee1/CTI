# Part 1: Configure ELK Stack on AWS

**ELK Stack Components**
- **Elasticsearch**
- **Logstash**
- **Kibana**

The ELK Stack (Elasticsearch, Logstash, and Kibana) is often extended with
**Beats** for lightweight data shipping. Together, they reliably and securely
ingest data from any source, in any format, then allow you to search, analyze,
and visualize it.

We can briefly summarize it this way:

* **Logstash**: Receives data (such as event logs) from various sources. It can
  parse, clean, and filter out the noise to extract useful information such as
  security telemetry, using the Elasticsearch Query Language (ES|QL).
* **Elasticsearch**: Stores parsed and normalized data from Logstash, enabling
  fast search and data analytics.
* **Kibana**: Provides a graphical interface (GUI) for data visualization and
  exploration.

Our telemetry can include security events collected in real time from our
laptop, which can then be ingested and analyzed using the ELK Stack.

[Read more](https://www.elastic.co/elastic-stack) for better understanding

Today, we're going to start by configuring Elasticsearch in our VPC (AWS).

---

## Configure Elasticsearch (AWS)

### 1. Create a VPC
- **Name**: SOC_lab
- **IPv4 CIDR**: `172.31.0.0/16`
- **Other options**: Leave as default

[Diagram](/VPC.png)

### 2. Create a Subnet in our VPC
- **VPC**: Select SOC_lab
- **Subnet name**: SOC_lab_LAN
- **Availability Zone**: Choose one (e.g., us-east-1a)
- **IPv4 subnet CIDR block**: `172.31.0.0/24`

### 3. Create an Internet Gateway
An internet gateway is a virtual router that connects a VPC to the internet.
- **Name tag**: SOC-internet-gateway
- **Action**: Attach to SOC_lab VPC

### 4. Create a Route Table
A route table specifies how packets are forwarded between the subnets within
your VPC, the internet, and your VPN connection.
- **Name**: SOC-route-table
- **VPC**: SOC_lab

**Edit Routes**:
- Add route: `0.0.0.0/0` → `SOC-internet-gateway`

**Associate with Subnet**:
- Go to **Subnet associations** tab
- Click **Edit subnet associations**
- Select **SOC_lab_LAN**
- Click **Save associations**

### 5. Create EC2 Instance
This is a virtual server in the cloud.

**Configuration**:
- **Name**: Elasticsearch-Server
- **OS Image**: Ubuntu Server 22.04 LTS
- **Instance type**: t3.micro (or t2.micro)
- **Key pair**: Create new pair for SSH (save the .pem file securely)

**Network Settings**:
- **VPC**: SOC_lab
- **Subnet**: SOC_lab_LAN
- **Auto-assign public IP**: Enable
- **Storage**: 8 GB gp3 (default)
- **Security Group**: Create new
  - **SSH (port 22)**: From "My IP" (recommended) or "Anywhere" (less secure)
  - **Note**: While "Anywhere" is acceptable for a temporary lab, always use "My IP" or AWS Systems Manager Session Manager for production environments.

My ISP dynamically allocates my IP. Therefore, I wasn't able to use "My IP"
setup.

- Click **Launch Instance**

### 6. Connect to your Ubuntu Instance
We wanna connect to our Ubuntu instance, download, configure and start
Elasticsearch service

- Connect remotely using your downloaded AWS SSH credentials
```Bash
# x.x.x.x - your instance public IP address
sudo ssh -i "SOC_ELK.pem" ubuntu@x.x.x.x
```
- Update & Upgrade your Ubuntu instance
```Bash
sudo apt-update
sudo apt-ugrade
```
- Download and Install Elasticsearch
```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-9.1.5-amd64.deb

dpkg -i elasticsearch-9.1.4-amd64.deb

```

[This](https://www.elastic.co/downloads/elasticsearch) is the official page for
Elasticsearch

- Configure Elasticsearch

```Bash
  # start a new shell with root privilege
  sudo -s
  # Check the elasticsearch config
  vi /etc/elasticsearch/elasticsearch.yml
```
We allowed connection from `http.host: 0.0.0.0` (anywhere) through the default
port, `9200`.
While "Anywhere" is acceptable for a temporary lab, always use your "Allocated
Public IP".

-  start Elasticsearch Service
```Bash
  sudo systemctl daemon-reload
  sudo systemctl enable elasticsearch.service
  sudo systemctl start elasticsearch.service
  # Elasticsearch service must be actively running
  sudo systemctl status elasticsearch.service
```
