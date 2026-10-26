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
The incident team reported that a user account was compromised through phishing.
Shortly thereafter an MFA token access, logged as metadata (webhook, api, json, etc...), was observed from an enterprise app.
Additional operational procedures noted that the organization admins performed auditing of app registrations to keep notes within "Branding & properties"; this is to help track business justifications, ownership, and investigation contexts.

1)  Identify Privilege Access Entrance
- My first lead was to find an indicator within app registration notes to help identify potential association with the compromised user account. 
- I navigated to App Registrations within the tenant and reviewed each application's "Branding & properties" blade.
- A note matching the compromised user account was found within a legacy application.
  
  <img width="1954" height="1263" alt="1-1 Navigation" src="https://github.com/user-attachments/assets/8794256e-aef8-4645-b33e-daa4e03ca45b" />


2)	Anticipate Privileged Escalation
- Further review within the "Certificate and secrets" blade hinted that the Owner privileges from the compromised user allowed the attacker to mint a new client secret for the legacy app's API until the expiry year of 2099.
- This minting of credentials (for almost a century) is suspiciously persistent, and would enable the attacker to inherit the legacy app's directory permissions (API permissions); providing a path to act as the Service Principal instead, no longer requiring authentication with the stolen token of the compromised user for future sign-ins.

  <img width="1954" height="1263" alt="2-1 Escalation Secret" src="https://github.com/user-attachments/assets/0b791846-0bff-4cdd-ab77-fc246046ea22" />

3)	Discover Attacker's Pivot of Delegated Access
- With this action of persistence, I decided to review the application's "Owners" blade: Confirming my suspicion of the attacker's actions to prevent remediation of rotating the secret(s).
- I found that the application owner(s) no longer listed the suspected compromised user account (as the "Branding & properties" notes implied), but instead listed the service principal of another application as the singular Owner instead (which was _not_ a legacy application's service principal).
- This discovery showed that the Owner has changed since the notes were applied, and could be a hint that another application's service principal was created; helping persist the attacker's privileged access through the legacy app by way of a newly created app.

  <img width="1954" height="1263" alt="3-1 Pivot App Secret and Ownership" src="https://github.com/user-attachments/assets/842a1140-e54d-41fd-a2ce-e4d69859111f" />

4)  Attacker's Privileged Peristance
- To confirm the blast radius of this new app, it's service principal, and the newly discovered ownership with this legacy application, I decided to review the "Expose an API" blade from within the legacy app.
- Found the legacy app's API exposed; this can be utilized as a callable method for other privileged service principals.  
5)  LOOT
- 


## What broke / what surprised me
- 

## Findings and recommendations

- 

## What I learned
- 
