# Create a Logical Diagram

The objective is to create a logical diagram that defines the workflow of any
project or lab before building it.

## Resources
* [Draw.io](https://draw.io/)

## Diagrams
* [Lab](/Lab_setup_diagram.jpg)

## Description

We will create and configure a Virtual Private Cloud (VPC) on AWS. We will set up
and configure:

- **Private Network**: Our machines will be assigned IP addresses to communicate
  locally within this range: `172.31.0.0/24` (172.31.0.1 – 172.31.0.254).
- **Windows Server (RDP Enabled)**: RDP (Remote Desktop Protocol) is a
  Microsoft-developed protocol that allows users to connect to and control a
  remote computer's graphical desktop over a network connection.
- **Fleet Server**: A component that connects Elastic Agents to Fleet.
- **osTicket Server**: Manages the ticketing system.
- **Ubuntu Server (SSH Enabled)**: SSH (Secure Shell) is a network protocol that
  enables secure, encrypted communication and remote access between computers
  over an insecure network.
- **Elastic & Kibana**: Elasticsearch indexes, analyzes, and searches ingested
  data, while Kibana visualizes the results of that analysis.
- **Default Gateway**: Connects our VPC to the internet.
- **C2 Server**: A simulated Command and Control (C2) server. A C2 server is a
  central system used by cybercriminals to control compromised devices and
  coordinate malicious activities, allowing them to manage attacks, exfiltrate
  data, and maintain persistent access to infected systems.

With this setup, we can:
- Simulate attacks
- Monitor, analyze, detect, and respond to threats and incidents
- Document our findings

