# The Privilege Audit (DRAFT)

### Auditing Privileged Access Across Azure

## Scenario

---

## Environment

* **Platform:** Microsoft Azure
* **Environment:** Live multi-user Azure training tenant
* **Access:** Reader access, plus one PIM-eligible role
* **Services reviewed:** Azure RBAC, Microsoft Entra ID, Privileged Identity Management (PIM)
* **Tools used:**

  * Access Control (IAM)
  * Azure CLI
  * Azure Resource Graph Explorer / KQL
  * Microsoft Entra Privileged Identity Management (PIM)
* **Audit focus:** Privileged role assignments, excessive access, orphaned assignments, standing vs. eligible privilege, and assignment scope

> **Note:** Sensitive lab values have been intentionally redacted. Challenge flags, RoleAssignmentDescription values, orphaned principal IDs, and other challenge-specific identifiers are not included.

---

# Scope and Methodology

## Methodology Comparison

| Method               | Sees                                        | Misses                              |
|----------------------|---------------------------------------------|-------------------------------------|
| IAM blade / export   | Active assignments at scope, inherited      | Group members, orphaned principals  |
| Azure CLI            | Same, plus null principalName (orphans)     | One scope per run                   |
| Resource Graph (KQL) | Whole tenant in one query                   | Eligible assignments                |
| PIM export           | Eligible vs active, activation history      | Assignments outside PIM             |

---

# Audit

## 1. Establishing an Access Baseline with IAM

### Method: Access Control (IAM) Blade

### What I Looked For

### What I Found

### Blind Spot

### Conclusion

### Evidence

#### Role Assignment Export

---

## 2. Auditing Role Assignments with Azure CLI

### Method: Azure CLI

### Command Used

```bash
az role assignment list --resource-group rg-<redacted>-prod-cus
```

---

### What I Looked For

---

### What I Found

---

### Why the Orphaned Assignment Matters

---

### Blind Spot

---

### Conclusion

---

### Evidence

#### Orphaned Role Assignment

---

## 3. Sweeping Role Assignments with Azure Resource Graph

### Method: Azure Resource Graph / KQL

### KQL Query

```kusto
authorizationresources
| where type =~ 'microsoft.authorization/roleassignments'
| extend principalId = tostring(properties.principalId)
| extend description = properties.description
| where principalId == '<REDACTED-ORPHANED-PRINCIPAL-ID>'
| project name,
          principalId,
          principalType = properties.principalType,
          scope = properties.scope,
          description
```

>

---

### What I Looked For


---

### What I Found

---

### Why Resource Graph Matters

---

### Blind Spot

---

### Conclusion


---

## 4. Auditing Eligible vs. Active Privilege with PIM


---

### What I Did

---

### What I Looked For

---

### What I Found

---

### Why Standing Privilege Matters

> 

to:

> 

---

### Activation History


---

### Blind Spot

> 

---

### Conclusion


---

### Evidence

#### PIM Role Assignments
