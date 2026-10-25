# Investigating a Suspicious Azure-Deployed Environment

## Scenario
An unauthorized user accessed a live Azure tenant within the previous 24 hours without triggering security alerts. The investigation focused on the identity plane to identify the initial access vector, compromised identity/application, and potential attacker persistence. Through forensic analysis of two linked app registrations, I reconstructed a five-stage OAuth consent-phishing kill chain, tracing the intrusion from consent abuse, token acquisition, unauthorized permissions, and retained access within the tenant.

## Environment

- **Platform**: Microsoft Azure
- **Environment**: Live multi-user Azure Training Tenant
- **Access Level**: Reader
- **Services Investigated**: Azure App Registrations, Tags, Policy Compliance, Policy Assignments
- **Governance Controls**: Azure Policy, Azure Resource Manager, Tags
- **Tools Used**: Azure Portal

## Investigation
The incident team reported that a user account was compromised through phishing, and shortly thereafter an MFA token access, logged as metadata (webhook, api, json, etc...), was observed from an enterprise app.
Additional operational procedures noted that the organization admins performed auditing of app registrations to keep notes within "Branding & properties"; this is to help track business justifications, ownership, and investigation contexts.

1)  Identify Privilege Access Entrance
- My first lead was to find an indicator within app registration notes to help identify potential association with the compromised user account. 
- I navigated to App Registrations within the tenant and reviewed each application's "Branding & properties" blade.
- A note matching the compromised user account was found within a legacy application.
  
  <img width="1954" height="1263" alt="1-1 Navigation" src="https://github.com/user-attachments/assets/8794256e-aef8-4645-b33e-daa4e03ca45b" />


2)	ESCALATE
- 
3)	PIVOT
- 
4)  PERSIST
- 
5)  LOOT
- 


## What broke / what surprised me
- 

## Findings and recommendations

- 

## What I learned
- 
