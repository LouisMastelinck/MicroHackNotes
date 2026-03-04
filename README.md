# MicroHackNotes



## Solution 1
```
### powershell
# =========================
# Setting commong variables
# =========================

# Customize RESOURCE_GROUP for each participant
$RESOURCE_GROUP = "labuser-03"   # Change this for each participant (e.g., labuser-01, labuser-02, ...)

$ATTENDEE_ID = $RESOURCE_GROUP
$SUBSCRIPTION_ID = "1c8e338e-802e-4d64-99d4-9a5a5ef469da"  # Replace with your subscription ID
$LOCATION = "norwayeast"  # If attending a MicroHack event, change to the location provided by your local MicroHack organizers

# =========================
# Generate friendly display names with attendee ID
# =========================

# Equivalent of Bash: ${ATTENDEE_ID#labuser-}
$AttendeeNumber = $ATTENDEE_ID -replace '^labuser-', ''

$DISPLAY_PREFIX = "Lab User-$AttendeeNumber"   # Converts "labuser-01" to "Lab User-01"
$GROUP_PREFIX   = "Lab-User-$AttendeeNumber"   # Converts "labuser-01" to "Lab-User-01"
```




Step 3 - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-3---option-b-assign-the-policy-using-azure-cli

```
$POLICY_NAME = "$ATTENDEE_ID-restrict-to-sovereign-regions"
$POLICY_DISPLAY_NAME = "$DISPLAY_PREFIX - Restrict to Sovereign Regions"
$POLICY_DEFINITION_ID = "e56962a6-4747-49cd-b67b-bf8b01975c4c"

# Policy parameters (PowerShell -> JSON)
$PolicyParams = @{
    listOfAllowedLocations = @{
        value = @(
            "norwayeast",
            "germanynorth",
            "northeurope"
        )
    }
} | ConvertTo-Json -Depth 5 -Compress

az policy assignment create `
  --subscription $SUBSCRIPTION_ID `
  --name $POLICY_NAME `
  --display-name $POLICY_DISPLAY_NAME `
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP" `
  --policy $POLICY_DEFINITION_ID `
  --params $PolicyParams
```




Step 4 - [https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-3---option-b-assign-the-policy-using-azure-cli](https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-4-also-restrict-resource-group-locations)


```
$RG_POLICY_DEFINITION_ID="e765b5de-1225-4ba3-bd56-1ac6695af988"

  az policy assignment create `
  --name "$ATTENDEE_ID-restrict-rg-to-sovereign-regions" `
  --display-name "$DISPLAY_PREFIX - Restrict Resource Groups to Sovereign Regions" `
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP" `
  --policy $RG_POLICY_DEFINITION_ID `
  --params $PolicyParams `
  --enforcement-mode DoNotEnforce
```



Step 1: Deny Public IP Address Creation - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-1-deny-public-ip-address-creation
```

# =========================
# Policy definition and names
# =========================

$DENY_RESOURCE_TYPE_POLICY = "6c112d4e-5bc7-47ae-a041-ea2d9dccd749"
$POLICY_NAME = "$ATTENDEE_ID-block-public-ip-addresses"
$POLICY_DISPLAY_NAME = "$DISPLAY_PREFIX - Block Public IP Addresses"

# =========================
# Policy parameters
# =========================

$PolicyParams = @{
    listOfResourceTypesNotAllowed = @{
        value = @(
            "Microsoft.Network/publicIPAddresses"
        )
    }
} | ConvertTo-Json -Depth 5 -Compress

# =========================
# Create policy assignment
# =========================

az policy assignment create `
  --name $POLICY_NAME `
  --display-name $POLICY_DISPLAY_NAME `
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP" `
  --policy $DENY_RESOURCE_TYPE_POLICY `
  --params $PolicyParams
```

Step 2 - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-2-enforce-private-endpoints-for-storage-accounts

```
az policy definition list `
  --query "[?displayName && contains(displayName, 'Storage') && contains(displayName, 'public')].{Name:displayName, ID:name}" `
  -o table
```

```
$STORAGE_PUBLIC_ACCESS_POLICY = "34c877ad-507e-4c82-993e-3452a6e0ad3c"
$POLICY_NAME = "$ATTENDEE_ID-storage-disable-public-access"
$POLICY_DISPLAY_NAME = "$DISPLAY_PREFIX - Storage accounts should disable public network access"


az policy assignment create `
  --name $POLICY_NAME `
  --display-name $POLICY_DISPLAY_NAME `
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP" `
  --policy $STORAGE_PUBLIC_ACCESS_POLICY
```


  Step 5 - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#task-5-create-a-policy-initiative-bonus

  Download this file: https://github.com/LouisMastelinck/MicroHackNotes/blob/main/Custom%20Initiative%20Definition.json
<img width="626" height="203" alt="image" src="https://github.com/user-attachments/assets/9b01375c-5ae0-4bd4-bd73-9ee3d6fa7b0e" />


step 2 - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-2-create-the-initiative-using-azure-cli
```
az policy set-definition create `
  --name "$ATTENDEE_ID-sovereign-cloud-baseline" `
  --display-name "$DISPLAY_PREFIX - Sovereign Cloud Security Baseline" `
  --description "Enforce location, tagging, and network controls for sovereign workloads" `
  --definitions sovereign-cloud-initiative.json `
  --subscription $SUBSCRIPTION_ID
```


Step 3 - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-3-assign-the-initiative
```
$INITIATIVE_ID = "/subscriptions/$SUBSCRIPTION_ID/providers/Microsoft.Authorization/policySetDefinitions/$($ATTENDEE_ID)-sovereign-cloud-baseline"

az policy assignment create `
  --name "$($ATTENDEE_ID)-sovereign-baseline-assignment" `
  --display-name "$($DISPLAY_PREFIX) - Sovereign Cloud Security Baseline" `
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP" `
  --policy-set-definition "$INITIATIVE_ID"
```


Task 6: Implement RBAC for SoverignOpsTeam

Step 1 - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-1-create-a-security-group-for-sovereignops-team
```
az ad group create `
  --display-name "$GROUP_PREFIX-SovereignOps-Team" `
  --mail-nickname "$GROUP_PREFIX-SovereignOps"

Start-Sleep -Seconds 20
```

step 2 - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-2-assign-built-in-roles-to-the-sovereignops-team
```
# Get the group's object ID
GROUP_OBJECT_ID=$(az ad group show --group "${GROUP_PREFIX}-SovereignOps-Team" --query id -o tsv)

# Assign Contributor role at resource group scope
# Get the group's object ID
$GROUP_OBJECT_ID = az ad group show `
  --group "$GROUP_PREFIX-SovereignOps-Team" `
  --query id `
  -o tsv

# Assign Contributor role at resource group scope
az role assignment create `
  --assignee $GROUP_OBJECT_ID `
  --role "Contributor" `
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP"
```
Task 7 - Create a Custom RBAC Role for Compliance Officers
Step 1: Create the Custom Role Definition - download this file: https://github.com/LouisMastelinck/MicroHackNotes/blob/main/compliance-auditor-role.json



https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#task-7-create-a-custom-rbac-role-for-compliance-officers
