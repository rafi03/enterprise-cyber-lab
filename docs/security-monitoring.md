# Security Monitoring Configuration

This guide details the setup and configuration of the Wazuh SIEM/XDR platform used for security monitoring in this lab environment.

## Table of Contents
- [Wazuh Overview](#wazuh-overview)
- [Deployment Architecture](#deployment-architecture)
- [Agent Deployment](#agent-deployment)
- [Agent Group Configuration](#agent-group-configuration)
- [Log Collection Configuration](#log-collection-configuration)
- [Custom Detection Rules](#custom-detection-rules)
- [Alert Configuration](#alert-configuration)
- [Dashboard Overview](#dashboard-overview)

## Wazuh Overview

Wazuh is an open-source security platform that provides extended detection and response (XDR) and Security Information and Event Management (SIEM) capabilities. It includes:

- Log data analytics
- Intrusion detection
- File integrity monitoring
- Vulnerability detection
- Compliance monitoring

## Deployment Architecture

![Wazuh Architecture](images/wazuh-architecture.png)

The deployment consists of:

1. **Wazuh Server (project-x-sec-box)**: Central component analyzing data from agents
2. **Wazuh Indexer**: Stores and indexes alerts
3. **Wazuh Dashboard**: Web UI for visualization and management
4. **Wazuh Agents**: Installed on endpoints to collect and forward data

## Agent Deployment

Agents have been deployed to all systems in the environment:

### Windows Agent Deployment

To deploy the Wazuh agent on Windows systems:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.9.2-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='10.0.0.10' WAZUH_AGENT_GROUP='Windows' WAZUH_AGENT_NAME='project-x-win-client'
```

Start the Wazuh agent service:
```powershell
NET START WAZUH
```

### Linux Agent Deployment

To deploy the Wazuh agent on Linux systems:

```bash
sudo wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.2-1_amd64.deb && sudo WAZUH_MANAGER='10.0.0.10' WAZUH_AGENT_GROUP='Linux' WAZUH_AGENT_NAME='project-x-linux-client' dpkg -i ./wazuh-agent_4.9.2-1_amd64.deb
```

Start the Wazuh agent service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

## Agent Group Configuration

Two agent groups have been created to manage configurations based on operating system:

1. **Windows Group**: Configuration for Windows systems
2. **Linux Group**: Configuration for Linux systems

![Agent Groups](images/wazuh-agent-groups.png)

## Log Collection Configuration

### Windows Log Collection

Windows Event Logs are collected from Security and Application channels:

```xml
<agent_config>
  <localfile>
    <location>Security</location>
    <log_format>eventchannel</log_format>
  </localfile>
  <localfile>
    <location>Application</location>
    <log_format>eventchannel</log_format>
  </localfile>
</agent_config>
```

### Linux Log Collection

Important security logs are collected from Linux systems:

```xml
<agent_config>
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/auth.log</location>
  </localfile>
  <localfile>
    <log_format>syslog</log_format>
    <location>/var/log/secure</location>
  </localfile>
  <localfile>
    <log_format>audit</log_format>
    <location>/var/log/audit/audit.log</location>
  </localfile>
</agent_config>
```

## Custom Detection Rules

Custom detection rules have been created for key security events:

### File Access Detection

The following rule detects access to sensitive files:

```xml
<group name="syscheck">
  <rule id="100002" level="10">
    <field name="file">secrets.txt</field>
    <match>modified</match>
    <description>File integrity monitoring alert - access to sensitive.txt file detected</description>
  </rule>
</group>
```

## Alert Configuration

Alerts have been configured for critical security events:

### Failed SSH Attempts Alert

![SSH Alert Configuration](images/ssh-alert-config.png)

Parameters:
- Trigger condition: Count > 2
- Severity: Medium (3)
- Time interval: 1 hour

### WinRM Logon Alert

![WinRM Alert Configuration](images/winrm-alert-config.png)

Parameters:
- Trigger condition: Count > 1
- Severity: Medium (3)
- Time interval: 1 hour

### Sensitive File Access Alert

![File Access Alert Configuration](images/file-access-alert.png)

Parameters:
- Trigger condition: Count > 1
- Severity: High (2)
- Time interval: 1 hour

## Dashboard Overview

The Wazuh dashboard provides comprehensive visibility into security events:

![Wazuh Dashboard](images/wazuh-dashboard.png)

Key dashboard components:
- **Security Events Overview**: Summary of all security events
- **Agent Status**: Health and connectivity of all agents
- **File Integrity Monitoring**: Detection of file changes
- **Alert Viewer**: Detailed information about triggered alerts
