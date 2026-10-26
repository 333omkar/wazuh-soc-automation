Wazuh SOC Automation Pipeline

A dockerized Wazuh SIEM lab with a custom brute-force detection rule, automated alert enrichment through VirusTotal, and automatic incident creation in ServiceNow, orchestrated with n8n.

Architecture

Overview

This project simulates a Tier 1 SOC workflow end to end: detect, enrich, and ticket, with no manual steps.

Detect: Wazuh correlates repeated failed logins using custom rule 100010, mapped to MITRE ATT&CK T1110 (Brute Force).
Collect: An n8n workflow queries the Wazuh Indexer every 5 minutes for new alerts.
Enrich: Source indicators from each alert are checked against the VirusTotal API.
Respond: n8n creates a ServiceNow incident containing the alert details and the enrichment verdict.
Architecture
Endpoint / Agent --> Wazuh Manager --> Wazuh Indexer
                                            |
                                  n8n (every 5 min)
                                            |
                                    VirusTotal API
                                            |
                                  ServiceNow Incident
Tech Stack
Component	Purpose
Wazuh (Docker)	SIEM: log collection, rules, alerting
Wazuh Indexer	Alert storage and search
n8n	Workflow automation and orchestration
VirusTotal API	Threat intelligence enrichment
ServiceNow	Incident management
Docker Compose	Deployment
Detection Rule
Field	Value
Rule ID	100010
Technique	T1110 - Brute Force
Logic	Correlates repeated authentication failures within a time window

The rule is in rules/local_rules.xml.

Repository Structure
.
├── docker-compose.yml
├── .env.example
├── config/            # Wazuh configuration (sanitized)
├── rules/             # Custom detection rules
├── n8n/               # Exported n8n workflow (sanitized)
└── docs/              # Architecture diagram and screenshots
Setup
Prerequisites
Docker Desktop (with Docker Compose)
A VirusTotal API key
A ServiceNow developer instance
n8n (self-hosted or cloud)
1. Clone the repo
bash
git clone https://github.com/333omkar/wazuh-soc-automation.git
cd wazuh-soc-automation
2. Configure environment variables
bash
cp .env.example .env

Edit .env and set your own values. Never commit this file.

3. Start Wazuh
bash
docker compose up -d

Open the Wazuh dashboard at https://localhost and sign in with the credentials you set.

4. Add the custom rule

Copy rules/local_rules.xml into the Wazuh manager's rules directory, then restart the manager:

bash
docker compose restart
5. Import the n8n workflow
Open n8n and choose Import from file.
Select n8n/workflow.json.
Add your credentials for the Wazuh Indexer, VirusTotal, and ServiceNow.
Activate the workflow.
Testing

Generate failed logins against a monitored endpoint to trigger rule 100010, then confirm:

 The alert appears in the Wazuh dashboard
 n8n picks it up on the next 5-minute run
 VirusTotal enrichment is attached
 A ServiceNow incident is created
Screenshots
Wazuh alert	n8n workflow	ServiceNow incident
alert	workflow	incident
Results

The pipeline was validated end to end, with brute-force alerts automatically producing enriched ServiceNow incidents (for example INC0010002 and INC0010003).

Security Notes
All credentials and API keys are supplied through environment variables.
No secrets, certificates, or private keys are stored in this repository.
This is a lab environment and is not hardened for production use.
Future Improvements
Add more correlation rules mapped to other MITRE ATT&CK techniques
Add Slack or email notifications
Add automated response actions (for example blocking source IPs)
Deploy on a cloud VM instead of a local machine
Author

Omkar Chavan LinkedIn | GitHub
