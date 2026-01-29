#SOC Automation Project 2.0 — How To Use AI in Your SOC Workflow

Overview

This project demonstrates an end-to-end SOC automation workflow where a Splunk alert triggers an n8n automation. n8n sends alert details to OpenAI (ChatGPT) for analysis (summary, MITRE ATT&CK mapping, severity, recommended actions), optionally enriches IOCs using AbuseIPDB, and posts the final structured output to Slack for the analyst to act on.

This is a portfolio/lab build. Not recommended for production as-is due to security telemetry being processed by an external AI API.

What I built (high-level flow)

Windows 10 generates security logs (failed logons)

Splunk ingests logs via Universal Forwarder

Splunk runs an alert (EventCode 4625 brute force)

Splunk webhook sends alert payload to n8n

n8n calls OpenAI to format + analyze the alert

n8n posts the result into a Slack channel

(Optional) n8n enriches source IP reputation using AbuseIPDB

Lab requirements
Hardware

Recommended: 32 GB RAM (works smoother)

Minimum: 16 GB RAM (may require turning off Kali/extra VMs)

Software

Hypervisor: VMware Workstation (VirtualBox also works)

VMs:

Windows 10 (endpoint)

Ubuntu Server (Splunk)

Ubuntu Server (n8n)

Optional: Kali Linux (attack simulation)

Accounts / API

Splunk account (download)

Slack account (workspace)

OpenAI API key + billing credits (minimum $5 is enough for lab testing)

Optional: AbuseIPDB API key

Step-by-step implementation
Step 1 — Create virtual machines

I used VMware Workstation and created these VMs:

1A) Windows 10 VM

vCPU: 2

RAM: 4 GB

Disk: 60 GB+

Network: NAT

1B) Ubuntu Server VM for Splunk

vCPU: 2

RAM: 8 GB

Disk: 100 GB (important for Splunk)

Network: NAT

Enabled OpenSSH Server during install (for SSH from host)

1C) Ubuntu Server VM for n8n

vCPU: 2

RAM: 4 GB

Disk: 50 GB

Network: NAT

Enabled OpenSSH Server

Optional: Kali VM

Used mainly for extra SOC telemetry generation. Not mandatory.

Step 2 — Install and configure Splunk Enterprise (on Ubuntu)
2A) Update Ubuntu packages

SSH into Splunk VM:

sudo apt-get update && sudo apt-get upgrade -y

2B) Download Splunk Enterprise (Linux .deb)

I logged into Splunk and downloaded Splunk Enterprise for Linux, then on the VM:

wget <splunk_deb_download_link>

2C) Install Splunk
sudo dpkg -i splunk*.deb

2D) Start Splunk and create admin user
sudo -u splunk /opt/splunk/bin/splunk start


Accepted license

Created Splunk admin username/password

2E) Enable Splunk to start on boot
sudo /opt/splunk/bin/splunk enable boot-start -user splunk

2F) Access Splunk Web

From host browser:

http://<splunk_vm_ip>:8000

Step 3 — Configure Splunk receiving + index + Windows Add-on
3A) Enable receiving port 9997

In Splunk Web:

Settings → Forwarding and receiving → Configure receiving → New receiving port

Set port: 9997

3B) Create dedicated index

Settings → Indexes → New Index

Name: dfir_project (or any name you prefer)

3C) Install Splunk Add-on for Microsoft Windows

Apps → Find More Apps

Install: Splunk Add-on for Microsoft Windows

Step 4 — Install Splunk Universal Forwarder on Windows 10
4A) Download Universal Forwarder

From Splunk download page:

Download Universal Forwarder for Windows 64-bit

4B) Install and point to Splunk receiver

During install:

Receiving Indexer: <splunk_vm_ip>

Port: 9997

4C) Create inputs.conf to enable event log collection

Path:
C:\Program Files\SplunkUniversalForwarder\etc\system\local\inputs.conf

Example inputs.conf (basic):

[default]
host = WIN10-DFIR

[WinEventLog://Security]
disabled = 0
index = dfir_project

[WinEventLog://System]
disabled = 0
index = dfir_project

[WinEventLog://Application]
disabled = 0
index = dfir_project

4D) Restart Splunk Forwarder service

Open Services (services.msc)

Restart SplunkForwarder

4E) Verify data is coming into Splunk

In Splunk Search:

index=dfir_project


I verified host + sources appeared (Security/System/Application logs).

Step 5 — Create a brute force detection alert in Splunk
5A) Generate failed logons

To generate EventCode 4625 quickly, I attempted RDP logins with wrong passwords multiple times.

5B) SPL query for failed logons
index=dfir_project EventCode=4625
| stats count by host user src_ip

5C) Save as Alert

Save As → Alert

Schedule: every minute (for testing)

Trigger action: Webhook

Webhook URL: (this comes from n8n in Step 7)

Step 6 — Install n8n on Ubuntu using Docker
6A) Update packages
sudo apt-get update && sudo apt-get upgrade -y

6B) Install Docker + Docker Compose
sudo apt-get install docker.io -y
sudo apt-get install docker-compose -y

6C) Create n8n docker-compose file
mkdir n8n-compose
cd n8n-compose
sudo nano docker-compose.yml


Example docker-compose.yml:

services:
  n8n:
    image: n8nio/n8n:latest
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=<n8n_vm_ip>
      - N8N_PORT=5678
      - N8N_PROTOCOL=http
    volumes:
      - ./n8n_data:/home/node/.n8n

6D) Fix permissions (common issue)
sudo chown -R 1000:1000 ./n8n_data

6E) Start n8n
sudo docker-compose up -d

6F) Access n8n Web

From host browser:

http://<n8n_vm_ip>:5678

Created n8n admin login.

Step 7 — Connect Splunk alert to n8n webhook
7A) Create Webhook workflow in n8n

New workflow → Add first node → Webhook

Method: POST

Copy the Test URL

7B) Paste webhook URL into Splunk alert action

Splunk Alert → Webhook action → paste URL → Save

7C) Test that Splunk sends data

In n8n:

Click Listen for test event
Splunk triggers → n8n receives payload (time, user, src_ip, count, etc.)

Step 8 — Add OpenAI (ChatGPT) analysis in n8n
8A) Add OpenAI node

Add node: OpenAI → Message a model

Paste OpenAI API key in credentials

Model: GPT (ex: GPT-4.1-mini or available option)

8B) Prompting (SOC Tier-1 assistant)

System / assistant prompt I used:

Summarize alert

Extract IOCs

Map to MITRE ATT&CK

Provide severity

Recommend next actions

Output in a structured format

8C) Pass webhook payload to OpenAI

I sent:

Alert name

JSON-stringified alert fields (so it works across alert types)

Step 9 — Send results to Slack
9A) Create Slack channel

Example: #alerts

9B) Create Slack app + token

Slack API → Create App → OAuth & Permissions

Added required scopes (chat:write, channels:read, etc.)

Installed app to workspace

Copied Bot OAuth Token

9C) Add Slack node in n8n

Node: Slack → Send message

Select channel: #alerts

Message content: output from OpenAI node

9D) Verify message appears

Slack started receiving formatted SOC alert analysis.

Step 10 (Optional) — Enrich IP with AbuseIPDB via HTTP Request tool
10A) Create AbuseIPDB API key

AbuseIPDB → API → Create key

10B) Add HTTP Request node in n8n

Use AbuseIPDB “check” endpoint

Pass the src_ip from alert

Feed results back into OpenAI prompt (so the model uses enrichment)

This improved the output by adding:

abuse confidence score

country / ISP context

reported categories (e.g., SSH brute force)

stronger response recommendations

Testing

I validated the workflow by:

generating 4625 failed logons

confirming Splunk indexed the events

triggering Splunk alert and confirming n8n webhook received data

confirming OpenAI produced structured SOC output

confirming Slack received the final analyst-ready summary

Conclusion

This project provides a comprehensive, hands-on experience in setting up and managing an Active Directory environment, integrating with Splunk for event logging and monitoring, and conducting security testing using Kali Linux and Atomic Red Team.

