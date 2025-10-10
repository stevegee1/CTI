# Part 2: Configure the ELK Stack on AWS

At this point, Elasticsearch is running on our Ubuntu server (port 9200).
Next, we’ll install and configure Kibana so we can visualize and interact with our Elasticsearch data.

## Step 1: Install Kibana

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-9.1.5-amd64.deb
sudo dpkg -i kibana-9.1.5-amd64.deb
```

Due to the limitation of our AWS Elastic IP, we’ll configure Kibana to listen on all interfaces (`0.0.0.0`) and use our AWS security group to restrict access.

Open the Kibana config file:

```bash
sudo vi /etc/kibana/kibana.yml
# Uncomment or add:
server.port: 5601
server.host: "0.0.0.0"
```

## Step 2: Configure AWS Security Group

Update the security group attached to your Ubuntu server instance:
- Allow inbound rules for TCP/5601 (Kibana) and TCP/9200 (Elasticsearch).
- Restrict access to your IP (or a limited set of trusted IPs).

Even though the services listen on all interfaces, the firewall ensures only
approved IPs can connect.

[Image](/Elastic_Kibana_sec_group.jpg)

## Step 3: Start Kibana

```bash
sudo systemctl daemon-reload
sudo systemctl enable kibana.service
sudo systemctl start kibana.service
# Elasticsearch service must already be running
sudo systemctl status kibana.service
```

## Step 4: Generate Enrollment Token

We need to connect Elasticsearch to Kibana. Although they run on the same server, they are separate services.
Generate an enrollment token for Kibana:

```bash
sudo -s
cd /usr/share/elasticsearch/bin
./elasticsearch-create-enrollment-token --scope kibana
# Copy and save the token
```

## Step 5: Access Kibana in Browser

Open Kibana in your browser:

```bash
http://<your-elastic-ip>:5601/
```

- Paste the enrollment token when prompted.
- Login with the default credentials:

```bash
Username: elastic
Password: <the superuser password generated for Elasticsearch in day 3>
```

## Step 6: Configure Encryption Keys

Kibana requires encryption keys to secure saved objects, reporting, and security features.

Generate keys:

```bash
sudo -s
cd /usr/share/kibana/bin
./kibana-encryption-keys generate
```

Add the generated keys:

```bash
./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey
./kibana-keystore add xpack.reporting.encryptionKey
./kibana-keystore add xpack.security.encryptionKey
```

Restart Kibana to apply changes:

```bash
systemctl restart kibana.service
```

## ✅ Done!

Your ELK Stack is now configured with Elasticsearch and Kibana running on AWS.
Access Kibana at: `http://<your-elastic-ip>:5601/`.

[Image](/kibana_GUI.png)
