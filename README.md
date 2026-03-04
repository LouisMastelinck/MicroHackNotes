# MicroHackNotes

### bash
## Global variables
# ##Set common variables
### Customize RESOURCE_GROUP for each participant
```
RESOURCE_GROUP="labuser-03"  # Change this for each participant (e.g., labuser-01, labuser-02, ...)

ATTENDEE_ID="${RESOURCE_GROUP}"
SUBSCRIPTION_ID="1c8e338e-802e-4d64-99d4-9a5a5ef469da"  # Replace with your subscription ID
LOCATION="norwayeast" #If attending a MicroHack event, change to the location provided by your local MicroHack organizers

# Generate friendly display names with attendee ID
DISPLAY_PREFIX="Lab User-${ATTENDEE_ID#labuser-}"  # Converts "labuser-01" to "Lab User-01"
GROUP_PREFIX="Lab-User-${ATTENDEE_ID#labuser-}"    # Converts "labuser-01" to "Lab-User-01"
```


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



# =========================
# Step 3 - https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-3---option-b-assign-the-policy-using-azure-cli
# =========================
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



# =========================
# Step 4 - [https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-3---option-b-assign-the-policy-using-azure-cli](https://github.com/microsoft/MicroHack/blob/main/03-Azure/01-03-Infrastructure/01_Sovereign_Cloud/walkthrough/challenge-01/solution-01.md#step-4-also-restrict-resource-group-locations)
# =========================

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

# =========================
# Create policy assignment
# =========================

az policy assignment create `
  --name $POLICY_NAME `
  --display-name $POLICY_DISPLAY_NAME `
  --scope "/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP" `
  --policy $STORAGE_PUBLIC_ACCESS_POLICY
```
  
