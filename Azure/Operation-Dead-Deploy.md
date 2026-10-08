# Investigating a Suspicious Azure-Deployed Environment

## Scenario
A junior intern was granted temporary contributor access to deploy a "test environment" within an Azure subscription, but the resulting resources were deployed outside the organization’s governance standards.
I investigated the environment with Reader access to determine what was deployed and how it was provisioned, with a focus on identifying why Azure Policy did not prevent the non-compliant deployment.

## Environment

- **Platform**: Microsoft Azure

- **Environment**: Live multi-user Azure Training Tenant

- **Access Level**: Reader

- **Services Investigated**: Azure Subscriptions, Resource Manager, Resource Groups, Tags, Resources, Initiatives/Policies, Policy Compliance, Policy Assignments

- **Governance Controls**: Azure Policy, Azure Resource Manager, Tags, Naming Standards

- **Tools Used**: Azure Resource Manager, ARM Deployments, Azure Policy, Azure Portal

## Investigation
I needed to observe and locate who, what, when, where, and how this governance violation occurred. My first methods are to reduce the scale and scope.
1)  I will begin by navigating to the Subscriptions and observe any anomalies among the resource groups. I may discover indicators that can help create a direction. Some of the biggest clues are whether there are any improper naming schemas, regions deployed, or conclusive tagging. It didn't take long to find "what" and "where" the outlier resided:
	<img width="1394" height="831" alt="1 - Navigation" src="https://github.com/user-attachments/assets/76f2e006-8acf-4385-9405-f8383145822b" />

2)	To confirm my suspicion, I visited the resource group to identify and review all resource(s) and accompanying tag(s).
-	Luckily, there was only one resource and viewing the tags provided evidence about the culprit and their name. Now I know the "who".
	<img width="1394" height="550" alt="2-1 - Resource Group - Resource" src="https://github.com/user-attachments/assets/de3668c7-bcf1-47f9-b2b0-6fbf303745d3" />
	<img width="1394" height="645" alt="2-2 - Resource Group - Resource - Tags" src="https://github.com/user-attachments/assets/0a4dc111-499d-41ed-809b-1f80dd195c8b" />

3)	Navigating backward through Azure Resource Manager, I can compare records against resource deployments.
-	I hopped to the resource group's deployment blade and it provided me details about this resource's conception: The timeline, parameters from the inputs, name of deployment, and successions. This told me the "what" that was provisioned and "when" this deployment occurred.
-	The fact that this deployment succeeded at all is the biggest clue to help lead me into investigating "how".
	<img width="1637" height="698" alt="3 - Resource Group - Resource - Deployment Blade" src="https://github.com/user-attachments/assets/cf40bd31-a4ef-4bf2-b21e-1b8c46dccd65" />

4)	Governance policies are supposed to enforce and prevent results like this. My only remaining answer could be within Azure tenant policies and confirm how these are configured and/or inherited.
- 	Investigating the resource group's policy overview showed multiple initiatives and a non-compliance graph, but I need to determine which ones by reviewing the compliance blade.
	<img width="1638" height="1270" alt="4-1 - Policy - Overview" src="https://github.com/user-attachments/assets/0b88cc57-7252-4dd5-9025-4e7e4a898ab9" />
- 	After closer inspection of the names, I found the non-compliant policy and confirmed that it's assigned to the correct resource group.
	<img width="1638" height="1270" alt="4-2 - Policy - Compliance" src="https://github.com/user-attachments/assets/2fb7cd7f-edf4-4400-a3fd-1137a7da3c84" />
- 	This particular policy's definition, along with the parameter for the "Effect", shows how this policy was inherited, but not enforced. This confirms "why" it was allowed.
	<img width="1638" height="1270" alt="4-3 - Policy - Compliance - Naming Convention" src="https://github.com/user-attachments/assets/da0556b9-6237-48ca-b3e6-30a678dee8b8" />

## What broke / what surprised me
-	**Losing my breadcrumb navigation:**
	Utilizing this helps me maintain navigation, but the page will reload seemingly randomly based on an object property or blade menu. This created an unintended effect that caused me to lose the convenience of retracing or maintaining my steps.
-	**Ensuring that I am reviewing the correct resource(s) or scope(s) after a page reload:**
  	After realizing the page reload could have navigated me away from the targeted resource, I find myself reviewing an entire page of information all over again. It's nice to know that I am aware and this training helps me stay acute to this behavior, but those page reloads currently cause me to re-assess often.
-	**Area of interest that I would utilize more frequently are the logs:**
  	This investigation was well-beyond 90 days for logs to be available. This is a bit expected for a large, controlled learning environment.

## Findings and recommendations
The investigation determined that the intern deployed a resource outside of this organization's naming standards based on a non-compliant policy effect.
-	Change the policy from "Audit" to "Deny" after validating resources against this policy. This will prevent future resources from violating the policy.
-	Review Governance requirements to ensure contributor access and deployment procedures are granularly defined.
-	Require appropriate tags at deployment for Subscriptions, Resource Groups, and Resources to help define their purposes (e.g. Purpose, Owners, Environments, Roles, Groups, Projects). This will help with future investigations and audits.

## What I learned
-	How to navigate Azure Portal with more familiarity and intent.
-	Sometimes the JSON preview can reveal more information than the details and property panes of resources.
-	Policies are often how these circumstances arise. Perhaps I will check those first, depending on the circumstances.
-	Tags are very useful with providing information at-a-glance, especially when other fields do not provide necessary information.
