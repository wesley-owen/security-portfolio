# Investigating a Suspicious Azure-Deployed Environment

## Scenario
An unauthorized user accessed a live Azure tenant within the previous 24 hours without triggering security alerts. The investigation focused on the identity plane to identify the initial access vector, compromised identity/application, and potential attacker persistence. Through forensic analysis of two linked app registrations, I reconstructed a five-stage OAuth consent-phishing kill chain, tracing the intrusion from consent abuse, token acquisition, unauthorized permissions, and retained access within the tenant.

## Environment

- **Platform**: Microsoft Azure
- **Environment**: Live multi-user Azure Training Tenant
- **Access Level**: Reader
- **Services Investigated**: Azure App Registrations, Tags, Policy Compliance, Policy Assignments
- **Governance Controls**: Azure Policy, Azure Identity, Tags
- **Tools Used**: Azure Portal

## Investigation
The incident team reported that a user account was compromised by stolen session (MFA token) through phishing.
Shortly thereafter an MFA token access, logged as metadata (webhook, api, json, etc...), was observed from an enterprise app.
Additional operational procedures noted that the organization admins performed auditing of app registrations to keep notes within "Branding & properties"; this is to help track business justifications, ownership, and investigation contexts.

1)  Identify Privilege Access Entrance
- My first lead was to find an indicator within app registration notes to help identify potential association with the compromised user account. 
- I navigated to App Registrations within the tenant and reviewed each application's "Branding & properties" blade.
- A note matching the compromised user account was found within a legacy application.
  <img width="1954" height="1263" alt="1-1 Navigation" src="https://github.com/user-attachments/assets/6fe5e9c0-4db3-4ea4-8e34-db7b099af1ee" />


2)	Escalation of Privileged Access Through Secrets
- Further review within the "Certificate and secrets" blade, there was a secret (credential) created with an expiration of the year 2099.
- A secret with the lifespan of any time longer than 2 years is already suspicious. This could only be allowed if the option of "Never Expire" was selected during the initial creation.
- This secret and expiration setting could have been created by our compromised user. However unlikely, we need to review the other configuration blades within this app to confirm there are no other indicators of increased access or attack-vectors.
  <img width="1954" height="1263" alt="2-1 Escalation Secret" src="https://github.com/user-attachments/assets/0b791846-0bff-4cdd-ab77-fc246046ea22" />

3)	Discover Attacker's Pivot of Delegated Access
- With this suspected action of persistence, I decided to review the application's "Owners" blade: Confirming my suspicion of the attacker's actions to evade remediation of rotating the secret(s).
- I found that the application owner(s) no longer listing the suspected compromised user account (as the "Branding & properties" notes implied), but instead listed the service principal of another registered app as the singular Owner.
- This discovery proves that the Owner has changed since the notes were applied, and could be proof that this other application was newly registered for ownership on behalf of the attacker; more than likely in an effort to help persist the attacker's privileged access by leveraging the legacy app's roles and permissions.
  <img width="1954" height="1263" alt="3-1 Pivot App Secret and Ownership" src="https://github.com/user-attachments/assets/842a1140-e54d-41fd-a2ce-e4d69859111f" />
  

4)  Attacker's Privileged Peristance
- To confirm the blast radius, I reviewed the "API Permissions" and "Expose an API" blade from within the legacy app.
- The legacy app's API was exposed; meaning this app can be utilized as a callable resource for other accounts (in this case, service accounts) to obtain delegated access. This is aligning with the new ownership and potential application registration.
  <img width="1954" height="1263" alt="4-1 Legacy App API Permissions" src="https://github.com/user-attachments/assets/4e8f79e9-1475-43bd-a621-0bbe41898b73" />
  <img width="1954" height="1263" alt="4-2 Persisting with Admin Granted API calls" src="https://github.com/user-attachments/assets/2a57d0ee-a2f9-46c0-9bab-a4d6afd0ad77" />

5)  Results
- Reviewing the newly registered app's service principal showed two Redirect URIs under the "Authentication" blade:
  - One pointing to a local-development port/address.
  - The other with a redirect URL value, containing an unrecognized domain destination.
- The redirect URI contained the same values as the exposed API that are scoped and callable from the legacy app.
- This proved that these two applications were being utilized for the purpose of stealing validated OAuth tokens and redirecting them to an attacker's infrastructure.
  <img width="1954" height="1263" alt="5-1 Suspicious Service Principal" src="https://github.com/user-attachments/assets/9c4df186-a681-4c61-a208-698b9d21d95f" />
  <img width="1954" height="1263" alt="5-1 Service Principal Redirect URI Authentication" src="https://github.com/user-attachments/assets/cf843fc5-107e-4dd5-8fb5-4be9de6f4989" />

## What broke / what surprised me
- My biggest surprise is how the attacker was allowed to register an application: This does not require any additional roles or permissions to perform by default in an Azure tenant. By default, Any user account within an Azure tenant can register an application.
- Accounts that register apps are designated as the application owner by default.

## Findings and recommendations
- Revoke and delete the compromised client secret.
- Remove the rogue service principal from the application's Owners list.
- Delete the custom Exposed API scope(s).
- Explicitly identify all relevant OAuth2PermissionGrant entries and revoke them.
- Remove the redirect URIs.
- Review and reduce Microsoft Graph's application permissions.
- Disable the default policy allowing users to register applications.
- Adjust privileged-access governance to audit control over applications registrations' Owners list, similar to user/admin roles.
- Deploy a detection rule, alerting when new client secrets and redirect URIs are created.
- Rotate any other potentially exposed credentials.

## What I learned
- OAuth2 separates "Authentication" versus "Authorization". Consent phishing abuses the Authorization portion after socially engineering the user that has already satisfied the Authentication.
- This is referred to as a "Confused Deputy" attack: A trusted, privileged service executes a request based on its own high-level permission without verifying the original caller's permissions.
- Application ownership does not implicate owner permissions. But in worse scenarios, the permissions could be adjusted by the owner. In cases like this where that application permissions are not scoped correctly, they are leveraged by another service to allow further insight and actions within the tenant.
- Results from trusted applications and consent phishing can be utilized not only for delegated permissions, but also application permissions.
- The legacy application, formerly owned by the user account, was provided permissive actions like "...Read.All" with Microsoft Graph, which would otherwise be inaccessible/unauthorized by the user's identity. Changing and removing the original owner with another service account made by the same compromised identity (and originating within the tenant) helped the attacker build a trusted, but malicious service through an exposed API.
