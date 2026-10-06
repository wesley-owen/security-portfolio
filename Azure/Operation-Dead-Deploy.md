# Investigating an Identity Attack in Entra ID - Draft

## Scenario
A junior intern was granted temporary contributor access to deploy a "test environment" within an Azure subscription, but the resulting resources were deployed outside the organization’s governance standards.
I investigated the environment with Reader access to determine what was deployed and how it was provisioned, with a focus on identifying why Azure Policy did not prevent the non-compliant deployment

## Environment
One list: platform, services, tools, access level. Honest framing: "live multi-user Azure training tenant, Reader access."

Platform: Microsoft Azure

Environment: Live multi-user Azure Training Tenant

Access Level: Reader

Services Investigated: Azure Subscriptions, Resource Groups, Resources, Policies, Tags, Policy Compliance, Policy Assignments, Initiatives, Resource Manager

Governance Controls: Azure Policy, Azure Resource Manager, Tags, Naming Standards

Tools Used: Azure Resource Manager, ARM Deployments, Azure Policy, Azure Portal



## Investigation
1. I needed to locate who, what, when, and how this violation occurred. My first methods are to reduce the scale and scope. I will begin with "What" by navigating to the Subscription's resource groups to observe any anomalies. I may discover indicators that can provide me with clues and narrow it from there. Some of the biggest clues are whether there is any improper naming schemas, regions deployed, or conclusive tagging. Fortunately, it didn't take long to find an indicator of "what" the resource group was.
<img width="1394" height="831" alt="1 - Navigation" src="https://github.com/user-attachments/assets/76f2e006-8acf-4385-9405-f8383145822b" />

2. To confirm my suspicion, I visited the resource group with intent to identify the resource(s) and review accompanying tag(s).
It appeared that viewing the tags provided evidence about the culprit and their name. Now that I know "who", I need to know "when" and "how" this made it through the Azure policies.
<img width="1394" height="550" alt="2-1 - Resource Group - Resource" src="https://github.com/user-attachments/assets/de3668c7-bcf1-47f9-b2b0-6fbf303745d3" />
<img width="1394" height="645" alt="2-2 - Resource Group - Resource - Tags" src="https://github.com/user-attachments/assets/0a4dc111-499d-41ed-809b-1f80dd195c8b" />

3. Navigating backward through Azure Resource Manager (ARM/JSON), I can compare records against resource deployments. I hopped to the resource group's deployment blade and it provided me details about this resource's conception: The timeline, parameters from the inputs, name of resource, and successions. This told me (again) what was provisioned, when this occurred, and could potentially provide a clue for "how" this was allowed. Unfortunately, not yet.
<img width="1637" height="698" alt="3 - Resource Group - Resource - Deployment Blade" src="https://github.com/user-attachments/assets/cf40bd31-a4ef-4bf2-b21e-1b8c46dccd65" />


4. Governance policies are supposed to enforce and prevent results like this. My only remaining answers at this time could be to reference the current Azure policies and confirm how they are configured and/or inherited. Investigating the resource group's policy overview showed multiple initiatives and a non-compliance graph, but I need to determine which ones by reviewing the compliance blade. After closer inspection of the names, I found the non-compliant policy that matters most and confirmed that it's assigned to the correct resource group. This particular policy's definition, along with the parameter for the "Effect", appears to be how this policy was inherited, but not enforced. This confirms "why" it was allowed.
<img width="1638" height="1270" alt="4-1 - Policy - Overview" src="https://github.com/user-attachments/assets/0b88cc57-7252-4dd5-9025-4e7e4a898ab9" />
<img width="1638" height="1270" alt="4-2 - Policy - Compliance" src="https://github.com/user-attachments/assets/2fb7cd7f-edf4-4400-a3fd-1137a7da3c84" />
<img width="1638" height="1270" alt="4-3 - Policy - Compliance - Naming Convention" src="https://github.com/user-attachments/assets/da0556b9-6237-48ca-b3e6-30a678dee8b8" />


## What broke / what surprised me
The most credible section in the document. Dead ends, wrong guesses, the thing that took an hour. Employers know real work is messy. This section separates you from certificate collectors.
The largest time consumers so far have been...
- Losing my breadcrumb navigation:
	Utilizing this helps me maintain navigation, but the page will reload seemingly randomly based on an object property or blade menu. This created an unintended effect that caused me to lose the convenience of retracing or maintaining my steps.
- Ensuring that I am reviewing the correct resource(s) or scope(s) after a page reload. I find myself reviewing an entire page of information all over again after taking it all in, then realizing the page reload could have navigated me away from the targeted resource. It's nice to know that I am aware and this training helps me stay acute to this behavior, but those page reloads currently cause me to re-assess often.
- Area of interest that I utilize more frequently are the logs, but the investigation was well-beyond 90 days for this to be available. This is a bit expected for a large, controlled learning environment.

## Findings and recommendations
TBD

## What I learned
TBD
