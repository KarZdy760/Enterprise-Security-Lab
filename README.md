# ENTERPRISE SECURITY LAB

Enterprise cybersecurity homelab built to simulate an enterprise environment for SOC/Blue Team operations, detection engineering, Windows Server and AD security. 

## Project Goals

The purpose of this project is to build a complete enterprise security environment from scratch and simulate real-world administration and security scenarios.

Current Infrastructure:

- Windows Server 2022 (Domain Controller)

- Windows 10 Client

- Docker Desktop

- Wazuh 4.14

- VMware Workstation

--

## Implemented Features

### Active Directory

- Domain Controller deployment

- Active Directory Domain Services configuration

- DNS server configuration

- Domain creation


### GPO Policy Hardening

- I implemented a set of Group Policy Object (GPO) policies to improve the security of the domain environment. The implemented policies included the configuration of Microsoft Defender, Windows Defender Firewall, password policies, Control Panel restrictions, screen lock settings, and other security-related configurations. Detailed screenshots of all implemented policies can be found in the SCREENS/GPO directory.

### Wazuh

- Wazuh Manager, Wazuh Indexer and Wazuh Dashboard deployed using Docker Compose

- Custom port configuration

- Domain Controller and W10 Client successfully enrolled as a Wazuh agent

## Current Goals

- Add Sysmon integration

- Add Custom detection rules

- Simulate attacks



