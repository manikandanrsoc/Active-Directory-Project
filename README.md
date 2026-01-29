# SOC Automation Project 2.0  
### How to Use AI in Your SOC Workflow

---

## 📌 Project Summary

**SOC Automation Project 2.0** is a hands-on, end-to-end Security Operations Center (SOC) automation lab that demonstrates how **SIEM detections can be enriched, analyzed, and operationalized using AI**.

In this project, I designed and implemented an **on-prem SOC workflow** where security alerts generated in **Splunk** are automatically sent to **n8n**, analyzed using **OpenAI (ChatGPT)**, enriched with external threat intelligence, and finally delivered to **Slack** in a structured, analyst-ready format.

This project is built purely for **learning and portfolio demonstration purposes** and showcases modern SOC concepts such as **SOAR-like automation, AI-assisted triage, and MITRE ATT&CK–aligned analysis**.

---

## 🎯 Objective

The primary goal of this project is to:
- Reduce manual alert triage effort in a SOC
- Demonstrate how AI can assist Tier-1 SOC analysts
- Automate alert summarization, enrichment, and response guidance
- Build a modern, relevant SOC portfolio project using real tools

---

## 🧠 What Problem This Solves

Traditional SOCs face:
- High alert fatigue  
- Inconsistent analysis quality  
- Time-consuming enrichment steps  
- Slow analyst response  

This project addresses these challenges by:
- Automatically processing alerts
- Standardizing analysis using AI
- Delivering clear, actionable output to analysts

---

## 🏗️ Architecture Overview


---

## 🧰 Technology Stack

- **SIEM:** Splunk Enterprise  
- **Automation:** n8n (Docker-based)  
- **AI:** OpenAI API (ChatGPT)  
- **Threat Intel:** AbuseIPDB API  
- **Endpoint:** Windows 10  
- **OS:** Ubuntu Server  
- **Messaging:** Slack API  
- **Containerization:** Docker & Docker Compose  

---

## 🔧 Key Features Implemented

### ✅ Log Collection & Detection
- Windows Security Event Logs ingested via Splunk Universal Forwarder
- Dedicated Splunk index for SOC project telemetry
- Detection use case: **Failed logon brute-force attempts (EventCode 4625)**

### ✅ Alert Automation
- Splunk alert configured with webhook action
- Alert payload sent automatically to n8n

### ✅ AI-Assisted Analysis
Using OpenAI, each alert is:
- Summarized in plain English
- Analyzed as a Tier-1 SOC analyst
- Mapped to **MITRE ATT&CK techniques**
- Assigned a severity rating
- Provided with recommended next actions

### ✅ Threat Intelligence Enrichment
- Source IP enriched using **AbuseIPDB**
- Confidence score, country, and abuse categories added
- AI uses enrichment context in final decision-making

### ✅ Analyst-Ready Output
- Final response posted to Slack
- Structured format:
  - Summary
  - Indicators of Compromise (IOCs)
  - Threat Intelligence
  - MITRE ATT&CK Mapping
  - Severity Assessment
  - Recommended Actions

---

## 🧪 How I Tested the Project

1. Generated failed RDP login attempts on Windows VM  
2. Verified logs were indexed in Splunk  
3. Confirmed Splunk alert triggered correctly  
4. Validated webhook payload reception in n8n  
5. Observed AI-generated analysis from OpenAI  
6. Confirmed structured SOC alert appeared in Slack  

---

## 🔐 Security & Privacy Considerations

- This lab processes security telemetry using an external AI API
- Not intended for production without:
  - Data masking/redaction
  - Access control hardening
  - Secure secret management
  - Legal/privacy review
- Demonstrates **concepts**, not production-ready deployment

---

## 🚀 Skills Demonstrated

- SOC operations & alert engineering  
- SIEM configuration and tuning (Splunk)  
- SOAR-style automation design  
- AI prompt engineering for security use cases  
- Threat intelligence integration  
- Docker-based service deployment  
- End-to-end incident workflow design  

---

## 📈 Future Enhancements

- DFIR-IRIS / Jira ticketing integration  
- VirusTotal hash enrichment  
- Automated response actions (block IP, disable user)  
- Local LLM deployment for sensitive environments  
- Additional detections (PowerShell abuse, persistence, malware execution)  

---

## 🧾 Resume-Ready Highlight

> Designed and implemented an AI-driven SOC automation pipeline using Splunk, n8n, and OpenAI to convert raw security alerts into MITRE-aligned, enriched, and actionable SOC intelligence delivered in real time to Slack.

---

## ⚠️ Disclaimer

This project is for **educational and portfolio purposes only**.  
Do not deploy this architecture in production without proper security, compliance, and privacy controls.

---

## 📂 Repository Structure (Recommended)

