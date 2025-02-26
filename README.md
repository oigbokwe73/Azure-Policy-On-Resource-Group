If you want to **iterate over the results** of your `az graph query` command in Azure CLI, you can do so using a **bash script** or **PowerShell**.

---

## **Using Bash (Linux/macOS/WSL)**
If you're using Azure CLI in a Bash environment, you can iterate over the results using a `for` loop.

### **Bash Script to Iterate Through Azure Resources with a Specific Tag**
```sh
#!/bin/bash

# Run the Azure Resource Graph query and store the output as JSON
resources=$(az graph query -q "Resources | where tags['Environment'] == 'Production'" --query "data" -o json)

# Loop through each resource
echo "$resources" | jq -c '.[]' | while read -r resource; do
    name=$(echo "$resource" | jq -r '.name')
    rg=$(echo "$resource" | jq -r '.resourceGroup')
    tags=$(echo "$resource" | jq -r '.tags')

    echo "Resource Name: $name"
    echo "Resource Group: $rg"
    echo "Tags: $tags"
    echo "--------------------------------"
done
```

> **Prerequisites:** Ensure `jq` is installed (`sudo apt install jq` on Ubuntu or `brew install jq` on macOS).

---

## **Using PowerShell (Windows/macOS/Linux)**
If you're using **PowerShell**, you can use the `ForEach-Object` loop.

### **PowerShell Script to Iterate Through Azure Resources with a Specific Tag**
```powershell
# Run Azure CLI query and store output as an object
$resources = az graph query -q "Resources | where tags['Environment'] == 'Production'" --query "data" | ConvertFrom-Json

# Iterate over each resource
foreach ($resource in $resources) {
    Write-Host "Resource Name: $($resource.name)"
    Write-Host "Resource Group: $($resource.resourceGroup)"
    Write-Host "Tags:"
    
    # Iterate through tags
    foreach ($tag in $resource.tags.PSObject.Properties) {
        Write-Host "  - $($tag.Name) : $($tag.Value)"
    }

    Write-Host "--------------------------------"
}
```

---

## **What These Scripts Do**
- Query **Azure Resource Graph** to filter resources by a specific tag (`Environment = Production`).
- Extracts relevant details (**name, resource group, tags**).
- Iterates over **each resource** and prints its details.
- Iterates over the **tags** associated with each resource.

Would you like to extend this to **filter by subscription** or **output to a CSV**? 🚀

Here's a **PowerShell script** that iterates through each resource in Azure and prints its name, resource group, and tags.

---

### **PowerShell Script to Iterate Through Each Resource with Tags**
```powershell
# Install and Import Az.ResourceGraph module if not installed
if (-not (Get-Module -ListAvailable -Name Az.ResourceGraph)) {
    Install-Module -Name Az.ResourceGraph -Force -AllowClobber
}

# Connect to Azure if not already authenticated
Connect-AzAccount

# Query to fetch all resources with tags
$query = @"
Resources
| where isnotempty(tags)
"@

# Run the query
$resources = Search-AzGraph -Query $query

# Iterate through each resource and print its details
foreach ($resource in $resources) {
    Write-Host "Resource Name: " $resource.name
    Write-Host "Resource Group: " $resource.resourceGroup
    Write-Host "Tags: "
    
    # Iterate through tags and print them
    foreach ($tag in $resource.tags.GetEnumerator()) {
        Write-Host "  - $($tag.Key) : $($tag.Value)"
    }

    Write-Host "----------------------------------------"
}
```

---

### **Explanation of the Script**
1. **Check & Install** `Az.ResourceGraph` module if it's not installed.
2. **Authenticate** using `Connect-AzAccount`.
3. **Query Azure Resource Graph** to get resources that have tags.
4. **Loop through each resource**:
   - Print the **resource name** and **resource group**.
   - Iterate through its **tags** and print the key-value pairs.

---

### **Output Example**
```
Resource Name:  MyVM1
Resource Group: RG-Production
Tags:
  - Environment : Production
  - Owner : DevOps Team
----------------------------------------
Resource Name:  SQLServer1
Resource Group: RG-Database
Tags:
  - Environment : Staging
  - Owner : Data Team
----------------------------------------
```

Would you like additional filtering, such as filtering by **subscription** or **resource type**? 🚀

You can use **Azure CLI** or **PowerShell** with the Microsoft Graph API to search for **Azure Tags**.

---

### **Using Azure CLI with Microsoft Graph API**
Azure CLI provides a direct way to query Azure Resource Graph for tags.

#### **CLI Script to Search for Azure Tags**
```sh
az graph query -q "Resources | where tags contains 'Environment'"
```
Replace `'Environment'` with the tag name you're searching for.

#### **CLI Script to Search for a Specific Tag and Value**
```sh
az graph query -q "Resources | where tags['Environment'] == 'Production'"
```
This finds all resources with the tag `Environment` set to `Production`.

---

### **Using PowerShell with Microsoft Graph API**
To retrieve tagged resources via PowerShell, you can use **Az.ResourceGraph**.

#### **PowerShell Script to Search for Azure Tags**
```powershell
# Install the Az.ResourceGraph module if not installed
Install-Module -Name Az.ResourceGraph -Force -AllowClobber

# Login to Azure
Connect-AzAccount

# Define the tag key you want to search for
$tagKey = "Environment"

# Define the query
$query = @"
Resources
| where tags['$tagKey'] != ''
"@

# Run the query
$resources = Search-AzGraph -Query $query

# Display the results
$resources | Format-Table name, resourceGroup, tags -AutoSize
```

#### **PowerShell Script to Search for a Specific Tag and Value**
```powershell
$tagKey = "Environment"
$tagValue = "Production"

$query = @"
Resources
| where tags['$tagKey'] == '$tagValue'
"@

$resources = Search-AzGraph -Query $query

$resources | Format-Table name, resourceGroup, tags -AutoSize
```

---

### **What These Scripts Do**
- Queries **Azure Resource Graph** to filter resources by tags.
- Uses the `tags` property to search for a specific tag or tag value.
- Outputs the matching resources in a table format.

Would you like additional filters (like filtering by subscription or resource type)? 🚀
Here's a **PowerShell script** to search for Azure tags across your Azure resources:

### **PowerShell Script to Search for Azure Tags**
```powershell
# Login to Azure if not already logged in
Connect-AzAccount

# Define the tag key and value to search for (Modify as needed)
$tagKey = "Environment"
$tagValue = "Production"

# Get all resources with the specific tag key and value
$resources = Get-AzResource | Where-Object { $_.Tags[$tagKey] -eq $tagValue }

# Display results
if ($resources) {
    $resources | Select-Object Name, ResourceGroupName, ResourceType, Location, @{Name="TagValue";Expression={$_.Tags[$tagKey]}} | Format-Table -AutoSize
} else {
    Write-Host "No resources found with the tag '$tagKey' = '$tagValue'"
}
```

---

### **Azure CLI Command to Search for Tags**
If you prefer **Azure CLI**, you can use the following command:
```sh
az resource list --tag Environment=Production --query "[].{Name:name, ResourceGroup:resourceGroup, Type:type, Location:location}" --output table
```
🔹 Replace `"Environment=Production"` with your desired tag key and value.

Would you like additional filtering options, such as searching within specific resource groups or subscriptions? 🚀






ea.portal.com was the legacy portal for managing Enterprise Agreements (EA) in Azure. It is now retired and replaced by the Cost Management + Billing feature within the Azure portal. You should use the Azure portal for all EA administration tasks now. [1, 1, 2, 2]  
Yes, you can use a Service Principal identity to create EA subscriptions. To do this, you need to grant the Service Principal appropriate permissions within your EA enrollment. [3, 4, 5]  
Here's a breakdown: [2, 6]  

• Enterprise Agreement (EA): This is a licensing program from Microsoft designed for large organizations to purchase and manage Microsoft products and services, including Azure, under a single agreement. [2, 2, 6, 6, 7, 8]  
• EA Enrollment: This represents your organization's commercial relationship with Microsoft and provides the billing foundation for your Azure subscriptions. [2, 2]  
• Service Principal: A security principal in Azure Active Directory (Azure AD) that represents an application or service. You can use a Service Principal to automate tasks, including creating subscriptions. [9, 9, 10, 10]  
• Granting Permissions: To allow a Service Principal to create EA subscriptions, an account owner or administrator with the necessary permissions (e.g., Owner role on the Enrollment Account) needs to assign appropriate roles to the Service Principal. [3, 3, 4, 4, 5, 5]  
• Using the Azure Portal (Cost Management + Billing): The Azure portal's Cost Management + Billing feature is where you manage your EA enrollment, including assigning roles to Service Principals and creating subscriptions. [1, 1, 2, 2, 11]  

Steps to use a Service Principal to create EA subscriptions: [3, 3, 4, 4, 10, 10]  

1. Create a Service Principal: You can create a Service Principal in Azure AD using the Azure portal or Azure CLI. [3, 3, 10, 10, 12]  
2. Grant Permissions: [3, 3, 5, 5]  
	• Sign in to the Azure portal as an account owner or administrator with the necessary permissions. [3, 5, 11]  
	• Navigate to Cost Management + Billing. [1, 3, 5, 11]  
	• Find your EA enrollment and access its settings. [3, 5, 13, 14]  
	• Assign the appropriate role to the Service Principal (e.g., "Azure Subscription Creator" or "Owner" on the appropriate billing scope). [3, 3, 5, 15, 16, 17]  


3. Use the Service Principal: You can now use the Service Principal (using tools like Azure CLI or PowerShell) to programmatically create new subscriptions under your EA. [3, 3, 4, 4, 18]  

Key Considerations: [3, 19]  

• API Version: Ensure you use the correct API version (e.g., 2019-10-01-preview) when working with EA role assignments and subscription creation. [3, 20, 21, 22]  
• Legacy APIs: While legacy APIs exist, it's recommended to use the latest GA version of the APIs for subscription creation unless you have a specific need to use the legacy ones. [5, 5]  
• Multiple EA Accounts: If you have multiple EA billing accounts, you must grant the necessary roles to the Service Principal in each account individually. [11, 11]  
• Automation: Using Service Principals enables automation of subscription creation, which can be beneficial for large organizations or DevOps scenarios. [3, 4, 5, 11, 23]  


Generative AI is experimental.

[1] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/direct-ea-administration[2] https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/landing-zone/design-area/azure-billing-enterprise-agreement[3] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/programmatically-create-subscription-enterprise-agreement[4] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/programmatically-create-subscription-preview[5] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/grant-access-to-create-subscription[6] https://www.ensono.com/insights-and-news/expert-opinions/microsoft-ea-vs-csp-which-right-you/[7] https://learn.microsoft.com/en-us/azure/cost-management-billing/microsoft-customer-agreement/onboard-microsoft-customer-agreement[8] https://licenseware.io/microsoft-enterprise-agreement-benefits-drawbacks-and-negotiation-tips/[9] https://docs.turbo360.com/docs/what-is-a-service-principal[10] https://docs.omnissa.com/bundle/HorizonCloudServicesUsingNextGenGuide/page/CreateaServicePrincipalfortheMicrosoftAzureSubscription.html[11] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/assign-roles-azure-service-principals[12] https://learn.microsoft.com/en-us/cli/azure/azure-cli-sp-tutorial-1[13] https://medium.com/cloudtechoffice/azure-ea-onboarding-guide-part-1-7820048da61c[14] https://help.ea.com/en/help/account/find-your-ea-account-email-address-using-your-game/[15] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/programmatically-create-subscription-microsoft-customer-agreement[16] https://github.com/Azure/azure-cli/issues/26470[17] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/add-change-subscription-administrator[18] https://www.damianflynn.com/blog/2020-04-24-delegating-ea-privilages-to-principal/[19] https://www.damianflynn.com/blog/2020-04-24-delegating-ea-privilages-to-principal/[20] https://microsoft.github.io/AzureTipsAndTricks/blog/tip213.html[21] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/programmatically-create-subscription-enterprise-agreement[22] https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/programmatically-create-subscription-enterprise-agreement[23] https://www.volt.io/content-hub/variable-recurring-payments-what-are-they-and-how-do-they-work/



### **Creating an Enterprise Agreement (EA) Subscription with a Service Principal in Azure**
Creating an **Enterprise Agreement (EA) Subscription** using a **Service Principal** requires configuring permissions and using the appropriate Azure APIs or automation tools.

---

## **Step 1: Ensure Prerequisites**
Before a Service Principal can create an EA subscription, you need:
- An **Enterprise Agreement (EA) Account**.
- **Enterprise Admin** or **Department Admin** permissions in the **EA Portal**.
- A **Service Principal (SPN) with the required role assignments**.
- Access to **Azure REST APIs** or **PowerShell/Azure CLI**.

---

## **Step 2: Assign Permissions to the Service Principal**
The Service Principal must be assigned **"Enrollment Account Contributor"** and **"Owner"** at the management group level.

### **1. Create a Service Principal (If Not Already Created)**
If you don’t have a Service Principal, create one using Azure CLI:

```bash
az ad sp create-for-rbac --name "EA-Subscription-SP" --role "Enrollment Account Contributor" --scopes "/providers/Microsoft.Billing/enrollmentAccounts/{enrollmentAccountId}"
```
- Save the **Client ID, Secret, and Tenant ID** for later use.

---

### **2. Assign Enterprise Admin Permissions**
1. **Sign in to the EA Portal**:  
   - Go to [EA Portal](https://ea.azure.com).
   - Navigate to **Manage > Enrollment Account**.

2. **Grant Access to the Service Principal**:
   - Find the **Enrollment Account ID**.
   - Assign the Service Principal as an **"Enrollment Account Contributor"**.

---

## **Step 3: Use REST API to Create the Subscription**
Once permissions are assigned, use the **Azure REST API** to create the EA subscription.

### **1. Obtain an Access Token**
Use the Service Principal credentials to obtain an OAuth token:

```bash
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" \
-d "grant_type=client_credentials&client_id={clientId}&client_secret={clientSecret}&resource=https://management.azure.com/" \
"https://login.microsoftonline.com/{tenantId}/oauth2/token"
```

Save the **access_token** returned.

---

### **2. Create the Subscription**
Use the **Microsoft.Subscription API** to create the new subscription:

```bash
curl -X POST "https://management.azure.com/providers/Microsoft.Subscription/createSubscription?api-version=2021-10-01" \
  -H "Authorization: Bearer {access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "displayName": "NewEA_Subscription",
    "billingScope": "/providers/Microsoft.Billing/enrollmentAccounts/{enrollmentAccountId}",
    "skuId": "0001",
    "subscriptionOwnerId": "/tenants/{tenantId}/users/{userId}"
  }'
```
- **displayName**: The name of the new subscription.
- **billingScope**: The EA Enrollment Account billing scope.
- **skuId**: Defines the subscription type.
- **subscriptionOwnerId**: The user or Service Principal that will own the subscription.

---

## **Step 4: Verify Subscription Creation**
Once the request is successful:
1. Navigate to the **Azure Portal** → **Subscriptions**.
2. Confirm the newly created EA subscription appears.
3. Assign additional users or Service Principals as needed.

---

## **Step 5: Assign Owner Permissions to the Subscription**
Since the subscription is new, you may need to assign **Owner** permissions:

```bash
az role assignment create --assignee <ServicePrincipalID> --role "Owner" --scope "/subscriptions/<subscriptionId>"
```

---

## **Optional: Automate Using PowerShell**
If you prefer PowerShell over REST API, you can use the `New-AzSubscription` cmdlet:

```powershell
Connect-AzAccount -ServicePrincipal -Tenant <tenantId> -ApplicationId <clientId> -CertificateThumbprint "<certThumbprint>"

New-AzSubscription -Name "EA_Subscription" -OfferType "MS-AZR-0017P" -BillingScope "/providers/Microsoft.Billing/enrollmentAccounts/{enrollmentAccountId}"
```

---

### **Key Considerations**
✅ **Permissions**: Service Principal must have the correct roles at the EA Billing Scope.  
✅ **API Quotas**: EA API limits how frequently new subscriptions can be created.  
✅ **Azure Policy Compliance**: Ensure the new subscription follows company policies.

Would you like a Terraform or Bicep template to automate this? 🚀



No, a **Service Principal (SP)** **cannot** create Azure subscriptions directly because subscription creation requires **billing account-level permissions**, which **cannot** be assigned to a service principal. 

### **Why a Service Principal Cannot Create Subscriptions?**
1. **Subscription Creation is a Billing Operation**:
   - Only users with **Enrollment Account Owner** or **Billing Account Owner** roles can create subscriptions.
   - These roles are assigned at the **billing scope** (not Azure RBAC), and **service principals are not supported** at this level.

2. **RBAC Limitations**:
   - Even if a service principal has **Owner** or **Contributor** access at the **Management Group or Tenant level**, it does **not** have permissions to create new subscriptions.

### **Workaround: Use a User Account with Automation**
Instead of a service principal, you can use:
1. **A User Account with EA (Enterprise Agreement) or MCA Billing Permissions**
2. **Azure Automation or PowerShell with Managed Identity** (if you can trigger an authenticated user session)

---
### **How to Automate Subscription Creation?**
Since a **Service Principal** cannot do this, you must use an authenticated user session:

#### **1. Using PowerShell (With User Login)**
```powershell
# Connect using an account with Enrollment Account Owner role
Connect-AzAccount

# Create a new subscription (requires EA permissions)
New-AzSubscription -OfferType "MS-AZR-0017P" -EnrollmentAccount "your-enrollment-account-email"
```

#### **2. Using Azure REST API with User Authentication**
If you want to automate this in CI/CD pipelines, you can use the **Azure REST API**, but it requires a **user account** with the right billing permissions:
```http
POST https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/{enrollmentAccountId}/providers/Microsoft.Subscription/createSubscription?api-version=2021-10-01
Authorization: Bearer <User Access Token>
Content-Type: application/json

{
  "displayName": "New Subscription",
  "offerType": "MS-AZR-0017P",
  "ownerId": "/providers/Microsoft.Billing/enrollmentAccounts/{enrollmentAccountId}/billingProfiles/{billingProfileId}"
}
```
✅ This **must be run with a user account**, not a Service Principal.

---
### **Alternative: Delegate Subscription Creation**
If automation is required, consider:
1. **Using a User Managed Identity (UMI)** in an **Azure Automation Runbook**.
2. **Using an admin user token in an Azure DevOps pipeline**.
3. **Manually creating subscriptions** and then using a service principal to manage them.

---
### **Final Answer**
🚫 **A Service Principal alone CANNOT create subscriptions.**
✅ **Use a User Account** with **Enrollment Account Owner** or **Billing Account Owner** roles.

Let me know if you need a deeper automation workaround! 🚀


## **Workaround: Automating Azure Subscription Creation Without Direct Service Principal Support**
Since a **Service Principal (SP)** **cannot create subscriptions** due to billing scope limitations, here are the best **workarounds** to automate subscription creation while maintaining security and compliance.

---

## **Workaround 1: Using Azure Automation Runbook with Managed Identity**
**Best for:** Fully automated subscription creation without requiring persistent user credentials.

### **Steps**
1. **Create an Azure Automation Account**
   - Go to **Azure Portal** → **Automation Accounts** → **+ Create**.
   - Select **Runbook Type: PowerShell**.
   - Assign a **User Managed Identity (UMI)** with the required billing permissions.

2. **Grant Billing Account Permissions to the Managed Identity**
   - **For EA (Enterprise Agreement)**: Assign the **Enrollment Account Owner** role to the UMI.
   - **For MCA (Microsoft Customer Agreement)**: Assign the **Billing Account Contributor** role.

3. **PowerShell Script in the Runbook**
   - This script authenticates the managed identity and creates a subscription.

   ```powershell
   # Connect to Azure using the managed identity
   Connect-AzAccount -Identity

   # Define subscription details
   $offerType = "MS-AZR-0017P" # Microsoft Azure Plan (Modify based on your need)
   $enrollmentAccount = "<your-enrollment-account-email>"
   $subscriptionName = "New-Azure-Subscription"

   # Create a new subscription
   New-AzSubscription -OfferType $offerType -EnrollmentAccount $enrollmentAccount -SubscriptionName $subscriptionName
   ```

4. **Schedule the Runbook** or trigger it using Azure Logic Apps.

---

## **Workaround 2: Using Azure DevOps Pipeline with User Authentication**
**Best for:** DevOps-driven automated subscription creation with controlled user access.

### **Steps**
1. **Create an Azure DevOps Pipeline**
   - Use **Azure CLI task** with **user authentication**.
   - Store a user account’s credentials securely in **Azure Key Vault** or **Azure DevOps Secret Variables**.

2. **Pipeline YAML Configuration**
   ```yaml
   trigger:
     - main

   jobs:
     - job: CreateSubscription
       displayName: 'Create Azure Subscription'
       pool:
         vmImage: 'ubuntu-latest'

       steps:
         - task: AzureCLI@2
           displayName: 'Login to Azure'
           inputs:
             azureSubscription: 'Your-Service-Connection'
             scriptType: 'bash'
             scriptLocation: 'inlineScript'
             inlineScript: |
               az login --service-principal -u $(SP_APP_ID) -p $(SP_SECRET) --tenant $(TENANT_ID)
               az account subscription create --billing-account-name "<billing-account-id>" --enrollment-account-name "<enrollment-account-id>" --display-name "NewSubscription"
   ```
3. **Store Credentials Securely**
   - Store **Client ID, Secret, and Tenant ID** in **Azure DevOps Secret Variables**.

---

## **Workaround 3: Using Azure REST API with an Authenticated User Session**
**Best for:** On-demand automation via API requests.

### **Steps**
1. **Obtain an Access Token**
   - Login using an **account with Enrollment Account Owner permissions**:
   ```bash
   az login --tenant <tenant_id> --scope https://management.azure.com/.default
   ```

2. **Make a REST API Call to Create a Subscription**
   ```bash
   curl -X POST "https://management.azure.com/providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountId>/providers/Microsoft.Subscription/createSubscription?api-version=2021-10-01" \
   -H "Authorization: Bearer <your_access_token>" \
   -H "Content-Type: application/json" \
   -d '{
     "displayName": "New Subscription",
     "offerType": "MS-AZR-0017P",
     "ownerId": "/providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountId>/billingProfiles/<billingProfileId>"
   }'
   ```

---

## **Workaround 4: Using Terraform with a User Account**
**Best for:** Infrastructure-as-Code (IaC) provisioning of new subscriptions.

### **Steps**
1. **Create a Terraform Configuration for Subscription**
   ```hcl
   provider "azurerm" {
     features {}
   }

   resource "azurerm_subscription" "example" {
     billing_scope_id = "/providers/Microsoft.Billing/enrollmentAccounts/<enrollmentAccountId>"
     display_name     = "New Subscription"
     subscription_billing_scope = "/providers/Microsoft.Billing/billingAccounts/<billingAccountId>"
   }
   ```
2. **Authenticate with an Admin User Account**
   ```bash
   terraform init
   terraform apply
   ```

---

## **Final Recommendation**
✅ **Best Option for Full Automation** → **Azure Automation Runbook with a Managed Identity**  
✅ **Best Option for DevOps Pipelines** → **Azure DevOps with User Credentials in Key Vault**  
✅ **Best Option for API-Based Automation** → **Azure REST API with an Authenticated User Session**  
✅ **Best for Infrastructure as Code** → **Terraform with User Authentication**  

Since **Service Principals cannot create subscriptions**, these workarounds ensure **secure, automated subscription creation** with **minimal manual intervention**. 🚀 Let me know if you need a detailed implementation for any!

Yes, to create Azure subscriptions using an **Enrollment Account**, you need **RBAC (Role-Based Access Control) permissions** at the **Azure Enterprise Agreement (EA) level** or in **Microsoft Cost Management + Billing** (for Microsoft Customer Agreement - MCA).

### **Steps to Create Azure Subscriptions with an Enrollment Account and Assign RBAC Access**
---

### **1. Verify Prerequisites**
Ensure you have the right permissions:
- **For EA (Enterprise Agreement) Accounts**:
  - You need to be an **Enterprise Administrator** or **Enrollment Account Owner**.
- **For MCA (Microsoft Customer Agreement)**:
  - You need **Billing Account Owner** or **Billing Profile Owner**.

### **2. Sign in to Azure Portal**
Go to **[Azure Portal](https://portal.azure.com)** and navigate to:
- **Cost Management + Billing** > **Billing scopes** > Select **Enrollment Account**.

### **3. Create a New Subscription**
1. In **Cost Management + Billing**, select **+ Add Subscription**.
2. Choose **Enrollment Account** or **Billing Profile**.
3. Select the desired **Subscription Offer** (Pay-As-You-Go, CSP, etc.).
4. Provide a **Subscription Name**.
5. Select the **Azure Region** and any associated policies.

### **4. Assign RBAC Permissions**
After the subscription is created, you must assign RBAC roles:
1. Go to **Azure Subscription** in **Azure Portal**.
2. Open **Access Control (IAM)**.
3. Click **+ Add Role Assignment**.
4. Select the role:
   - **Owner** – Full access including billing.
   - **Contributor** – Can manage resources but not assign roles.
   - **Reader** – View-only access.
5. Select the **User, Group, or Service Principal**.
6. Click **Save**.

### **5. Validate Subscription and Access**
- Go to **Subscriptions** in the Azure portal.
- Ensure the correct users have access under **IAM**.

---
### **Automation Using Azure PowerShell**
If you need to automate subscription creation, use:

```powershell
# Login to Azure
Connect-AzAccount

# Create a new subscription
New-AzSubscription -OfferType "MS-AZR-0017P" -EnrollmentAccount "your-enrollment-account"

# Assign RBAC permissions
New-AzRoleAssignment -Scope "/subscriptions/{subscriptionId}" -RoleDefinitionName "Owner" -PrincipalId "{User_Object_ID}"
```
---
This ensures that your **Azure Subscription is created** with proper **RBAC access** via an Enrollment Account. Let me know if you need additional details! 🚀
You're correct that Terraform does not directly support creating **Azure Subscriptions** in the current AzureRM provider version (as of now). Azure subscription creation requires interacting with the **Azure Subscription API**, which Terraform does not natively manage. However, you can automate this process using Azure CLI, custom scripts, or an external tool in conjunction with Terraform.

Below is a workaround to achieve this:

---

### Alternative Method: Use Azure CLI to Create the Subscription
You can use the **Azure CLI** to create a subscription and then use Terraform to manage resources under it.

#### Step 1: Create the Subscription with Azure CLI
Run the following Azure CLI command to create a new subscription under a billing account:

```bash
az account subscription create \
    --billing-account-name "<Billing_Account_Name>" \
    --billing-profile-name "<Billing_Profile_Name>" \
    --invoice-section-name "<Invoice_Section_Name>" \
    --display-name "Example Subscription"
```

Replace:
- `<Billing_Account_Name>` with your billing account name.
- `<Billing_Profile_Name>` with the billing profile in your billing account.
- `<Invoice_Section_Name>` with the invoice section under the billing profile.

#### Step 2: Use Terraform to Manage Resources in the Subscription
After creating the subscription, fetch its ID and use Terraform to manage resources in it.

1. **Set the Subscription ID**:
   Update your Terraform configuration with the new subscription ID:
   ```hcl
   provider "azurerm" {
     features {}
     subscription_id = "<New_Subscription_ID>"
   }
   ```

2. **Terraform Script to Create a Resource Group**:
   ```hcl
   provider "azurerm" {
     features {}
     subscription_id = var.subscription_id
   }

   resource "azurerm_resource_group" "example" {
     name     = "example-resource-group"
     location = "East US"
     tags = {
       Environment = "Development"
     }
   }

   output "resource_group_id" {
     value = azurerm_resource_group.example.id
   }
   ```

   Define `var.subscription_id` in your `variables.tf`:
   ```hcl
   variable "subscription_id" {}
   ```

   Then pass the new subscription ID:
   ```bash
   terraform apply -var="subscription_id=<New_Subscription_ID>"
   ```

---

### Full Automation: Using Azure REST API in Terraform

You can integrate the **Azure REST API** to create subscriptions. Here's an example using Terraform's `http` provider to call Azure's Subscription API:

#### Example Script:
```hcl
provider "azurerm" {
  features {}
}

data "azurerm_client_config" "current" {}

resource "null_resource" "create_subscription" {
  provisioner "local-exec" {
    command = <<EOT
    az rest --method post \
    --url "https://management.azure.com/providers/Microsoft.Billing/billingAccounts/<Billing_Account_ID>/providers/Microsoft.Subscription/createSubscription?api-version=2020-10-01" \
    --body '{
      "displayName": "Example Subscription",
      "skuId": "0001",  # Replace with a valid SKU ID
      "billingScope": "/providers/Microsoft.Billing/billingAccounts/<Billing_Account_ID>/billingProfiles/<Billing_Profile_ID>"
    }'
    'EOT
  }
}
```

#### Notes:
- Replace `<Billing_Account_ID>` and `<Billing_Profile_ID>` with actual values.
- This script uses the Azure CLI (`az rest`) to call the Subscription API.

---

### Conclusion:
Terraform cannot natively create subscriptions, but you can:
1. Use **Azure CLI** to create a subscription and then use Terraform to manage its resources.
2. Use the **Azure REST API** within Terraform (with custom provisioning).
3. Use external automation tools to combine Terraform with subscription creation.

Would you like more details on automating subscription creation with REST APIs?


Here’s an example of using the `<<EOT` syntax in Terraform to execute a multi-line command, specifically for creating a subscription using the **Azure REST API** via the Azure CLI.

---

### Example Script: Creating a Subscription with Azure REST API in Terraform

```hcl
provider "azurerm" {
  features {}
}

resource "null_resource" "create_subscription" {
  provisioner "local-exec" {
    command = <<EOT
    az rest --method post \
    --url "https://management.azure.com/providers/Microsoft.Billing/billingAccounts/<Billing_Account_ID>/providers/Microsoft.Subscription/createSubscription?api-version=2020-10-01" \
    --body '{
      "displayName": "Example Subscription",
      "skuId": "0001",  # Subscription SKU (e.g., Pay-As-You-Go)
      "billingScope": "/providers/Microsoft.Billing/billingAccounts/<Billing_Account_ID>/billingProfiles/<Billing_Profile_ID>"
    }' \
    --headers "Content-Type=application/json"
    EOT
  }
}

output "subscription_creation_status" {
  value = "Subscription creation initiated. Please verify in the Azure portal."
}
```

---

### Explanation of Key Elements:
1. **Provisioner `local-exec`**:
   - Executes a shell command during the Terraform deployment process.

2. **`<<EOT` Syntax**:
   - Used for multi-line strings in Terraform, making it easier to include complex scripts or JSON payloads.

3. **Azure CLI Command**:
   - **`az rest`**: Calls the Azure REST API directly from the CLI.
   - **`--method post`**: Specifies the HTTP method for the REST call.
   - **`--url`**: Points to the Azure Subscription API endpoint.
   - **`--body`**: Supplies the payload in JSON format, which contains the details of the subscription to be created.
   - **`--headers`**: Adds the `Content-Type` header to specify the request body format.

4. **Variables to Replace**:
   - `<Billing_Account_ID>`: The ID of your Azure billing account.
   - `<Billing_Profile_ID>`: The billing profile ID within your billing account.

---

### How to Run:
1. Replace `<Billing_Account_ID>` and `<Billing_Profile_ID>` with actual IDs from your Azure billing account.
2. Save the script to a Terraform file, e.g., `main.tf`.
3. Run the following Terraform commands:
   ```bash
   terraform init
   terraform apply
   ```

---

### Example REST API Payload:
Here's an example JSON body sent in the REST API call:
```json
{
  "displayName": "Example Subscription",
  "skuId": "0001",
  "billingScope": "/providers/Microsoft.Billing/billingAccounts/<Billing_Account_ID>/billingProfiles/<Billing_Profile_ID>"
}
```

---

### Additional Notes:
- After running this script, the subscription creation request is sent, but provisioning the subscription may take some time. Check the Azure portal for updates.
- Ensure the Service Principal or user running the Terraform script has appropriate permissions for the Billing Account.

Would you like help retrieving your `<Billing_Account_ID>` or `<Billing_Profile_ID>`?
### **Creating a Pay-As-You-Go Subscription Using Terraform & Microsoft Graph API**
Since **Azure does not support creating a Pay-As-You-Go subscription via Terraform or the Azure CLI**, the workaround is to use **Microsoft Graph API** with Terraform to automate subscription creation.

---

## **🚀 Steps to Create a Pay-As-You-Go Subscription Using Terraform & Graph API**
1. **Register an App in Microsoft Entra ID (Azure AD)**
2. **Grant API Permissions to the App**
3. **Use Terraform to Call Microsoft Graph API** to create the subscription.

---

### **🛠 Step 1: Register an App in Microsoft Entra ID**
1. Navigate to [Microsoft Entra Admin Center](https://entra.microsoft.com).
2. Go to **App registrations > New registration**.
3. Give it a name (e.g., `"SubscriptionCreatorApp"`).
4. Choose **Accounts in this organizational directory only**.
5. Click **Register**.
6. Note down the **Application (Client) ID** and **Directory (Tenant) ID**.

---

### **🔑 Step 2: Assign API Permissions**
1. Open your registered app.
2. Go to **API Permissions > Add Permission**.
3. Select **Microsoft Graph > Application Permissions**.
4. Search and add:
   - `Subscription.ReadWrite.All`
   - `Directory.Read.All`
   - `Billing.ReadWrite`
5. Click **Grant admin consent**.

---

### **🔐 Step 3: Create a Client Secret**
1. Navigate to **Certificates & Secrets**.
2. Click **New client secret**.
3. Copy the **secret value** (it will be needed in Terraform).

---

### **📌 Step 4: Create a Terraform Configuration to Call Microsoft Graph API**
Now, we use Terraform’s `http` provider to make an API request to **Microsoft Graph** to create a new subscription.

#### **🌍 Terraform Configuration**
```hcl
provider "azuread" {
  tenant_id = "<your-tenant-id>"
}

provider "http" {}

resource "null_resource" "create_subscription" {
  provisioner "local-exec" {
    command = <<EOT
    curl -X POST https://graph.microsoft.com/v1.0/subscriptions \
      -H "Authorization: Bearer $(az account get-access-token --resource https://graph.microsoft.com --query accessToken -o tsv)" \
      -H "Content-Type: application/json" \
      -d '{
        "displayName": "My Pay-As-You-Go Subscription",
        "skuId": "0001",
        "billingAccountId": "/providers/Microsoft.Billing/billingAccounts/{billingAccountId}",
        "invoiceSectionId": "/providers/Microsoft.Billing/billingAccounts/{billingAccountId}/invoiceSections/{invoiceSectionId}"
      }'
    EOT
  }
  
  triggers = {
    always_run = timestamp()
  }
}
```

---

### **🔎 Explanation**
- This Terraform script **calls Microsoft Graph API** to create a new Azure **Pay-As-You-Go subscription**.
- `skuId: "0001"` represents **Pay-As-You-Go**, but you need to confirm this in your billing account.
- `{billingAccountId}` and `{invoiceSectionId}` must be replaced with actual **billing details** (found in **Azure Cost Management & Billing**).

---

### **🚀 Running the Terraform Script**
```sh
terraform init
terraform apply -auto-approve
```

---

### **🛠 Debugging & Verification**
To check if the subscription was created:
```sh
az account list --output table
```
or
```sh
az graph query -q "Resources | where type == 'microsoft.billing/subscriptions'"
```

---

### **🎯 Final Notes**
✅ **Terraform + Graph API can automate Pay-As-You-Go subscription creation** for **Enterprise Agreement (EA) and CSP customers**.  
❌ **Pay-As-You-Go cannot be created via Terraform for individual users**—it must be done via the Azure Portal.  
💡 **Billing accounts must be pre-configured** for API-based subscription creation.  

Would you like me to refine the Terraform script based on your **specific billing details**? 😊


The `null_resource` resource in Terraform is used to define actions or dependencies that are not directly tied to a specific cloud resource but still need to be managed within your Terraform configuration. A common use case for `null_resource` is to execute provisioning logic using `local-exec` or `remote-exec` provisioners or to create dependencies between resources that are otherwise not linked.

### Example: Creating a Subscription with `null_resource`

If you're using `null_resource` to trigger an action, like creating a subscription or running a script, here's an example setup:

```hcl
resource "null_resource" "create_subscription" {
  provisioner "local-exec" {
    command = <<EOT
    az account subscription create \
      --name "MySubscription" \
      --offer-type "MS-AZR-0123P" \
      --subscription-id "<your-subscription-id>"
    EOT
  }

  triggers = {
    always_run = timestamp() # Ensures it always runs on change
  }
}
```

### Key Points:
1. **Provisioner (`local-exec`)**:
   - Executes a local command to create a subscription using Azure CLI.
   - Replace the `command` value with the actual script or CLI command to create the subscription.

2. **Triggers**:
   - Use the `triggers` block to specify conditions under which the `null_resource` will re-run. 
   - Example: `timestamp()` ensures it always executes when you apply the configuration.

3. **Dependencies**:
   - You can use the `depends_on` argument to define dependencies on other resources to ensure this runs after they are created.

### Example with Dependencies:
If creating a subscription depends on some other Azure resource, you can add:

```hcl
resource "null_resource" "create_subscription" {
  provisioner "local-exec" {
    command = "az account subscription create --name 'MySubscription' --offer-type 'MS-AZR-0123P'"
  }

  triggers = {
    resource_group_id = azurerm_resource_group.example.id
  }

  depends_on = [azurerm_resource_group.example]
}
```

This ensures that the subscription is created only after the `azurerm_resource_group` is provisioned.

Would you like help integrating this with other Azure Terraform resources or custom logic?


Here’s a PowerShell script to create a new subscription and add it to a management group in Azure. It includes commands to locate the billing account name dynamically. Make sure you have the required permissions to perform these actions.


Creating a new Azure subscription using Terraform involves leveraging the Azure Management Groups and the Azure Subscription API. Below is a detailed Terraform script to create a new Azure subscription and associate it with a Management Group.

### Prerequisites:
1. Ensure you have the required permissions to create Azure subscriptions (e.g., "Owner" or "Contributor" role at the tenant level).
2. Install Terraform CLI.
3. Set up the Azure CLI and authenticate to your Azure account.
4. Register the `Microsoft.Subscription` resource provider in your Azure tenant.

### Terraform Script

This script provisions a new subscription under a specified management group.

```hcl
provider "azurerm" {
  features {}
  subscription_id = "<Management_Subscription_ID>" # Replace with the ID of the management subscription
}

resource "azurerm_client_config" "current" {}

# Define the management group where the subscription will be associated
resource "azurerm_management_group" "example" {
  name        = "example-management-group"
  display_name = "Example Management Group"
}

# Create a new Azure subscription
resource "azurerm_subscription" "new_subscription" {
  subscription_name   = "Example Subscription"  # Name of the new subscription
  billing_scope_id    = "<BillingScopeID>"      # Replace with your billing account ID
  management_group_id = azurerm_management_group.example.id
  subscription_alias  = "example-subscription-alias"
}

# Output the new subscription details
output "new_subscription_id" {
  value = azurerm_subscription.new_subscription.id
}

output "new_subscription_alias" {
  value = azurerm_subscription.new_subscription.subscription_alias
}
```

### Explanation of the Script:
1. **Provider Configuration**: 
   - The `azurerm` provider is configured with a subscription ID that has permissions to create subscriptions.

2. **Management Group**:
   - Creates or references a management group under which the new subscription will be added.

3. **Subscription Resource**:
   - The `azurerm_subscription` resource is used to create a new Azure subscription. You need to specify the `billing_scope_id` from your Azure billing account.

4. **Outputs**:
   - Outputs the `subscription_id` and `subscription_alias` for reference after the subscription is created.

### Steps to Execute:
1. Save the script to a file, e.g., `main.tf`.
2. Initialize Terraform in the script's directory:
   ```bash
   terraform init
   ```
3. Review the execution plan:
   ```bash
   terraform plan
   ```
4. Apply the configuration to create the subscription:
   ```bash
   terraform apply
   ```

### Notes:
- Replace `<Management_Subscription_ID>` with the ID of the subscription that manages your Azure resources.
- Replace `<BillingScopeID>` with your Azure billing account ID. You can find this in the Azure portal under **Cost Management + Billing**.
- Ensure the management group name (`example-management-group`) is unique or corresponds to an existing group in your environment.

Would you like a diagram to visualize this setup?

---

### **PowerShell Script**

```powershell
# Log in to Azure
Connect-AzAccount

# Variables - Update these values
$SubscriptionDisplayName = "NewSubscriptionName"  # Set your desired subscription name
$ManagementGroupName = "ManagementGroupName"     # Set your target management group name

# Step 1: Retrieve the Billing Account Name
Write-Host "Retrieving billing account information..."
$BillingAccount = Get-AzBillingAccount
if ($BillingAccount.Count -eq 0) {
    Write-Host "No billing accounts found. Ensure you have sufficient permissions." -ForegroundColor Red
    exit
}
Write-Host "Billing Account Name: " $BillingAccount[0].Name

# Step 2: Create a New Subscription
$BillingAccountName = $BillingAccount[0].Name
$NewSubscription = New-AzSubscription -BillingAccountName $BillingAccountName -DisplayName $SubscriptionDisplayName

if ($NewSubscription -eq $null) {
    Write-Host "Failed to create subscription. Ensure proper permissions and inputs." -ForegroundColor Red
    exit
}

Write-Host "Subscription Created: " $NewSubscription.SubscriptionId

# Step 3: Add the Subscription to a Management Group
Write-Host "Adding subscription to the management group..."
$SubscriptionId = $NewSubscription.SubscriptionId
Add-AzManagementGroupSubscription -GroupId $ManagementGroupName -SubscriptionId $SubscriptionId

Write-Host "Subscription successfully added to management group: $ManagementGroupName" -ForegroundColor Green
```

---

### **Prerequisites**
1. **Azure PowerShell Module**:
   - Ensure you have the **Azure PowerShell Module** installed.
   - To install, run:
     ```powershell
     Install-Module -Name Az -AllowClobber -Scope CurrentUser
     ```
   - Import it if needed:
     ```powershell
     Import-Module Az
     ```

2. **Permissions**:
   - **Create Subscription**: Requires permissions on the billing account (e.g., **Owner**, **Contributor**).
   - **Management Group**: Requires **Owner** or **Contributor** on the management group.

---

### **What It Does**
1. Connects to your Azure account.
2. Retrieves the billing account name dynamically.
3. Creates a new subscription under the billing account.
4. Adds the subscription to the specified management group.

---

Let me know if you encounter any issues or need further modifications!


Below is an example of a Terraform module to create an Azure subscription and retrieve its output:

### Pre-requisites
- Ensure you have permissions to create Azure subscriptions.
- Terraform and Azure CLI are properly installed and configured.

### Terraform Module for Azure Subscription
```hcl
# Create a new directory for the module and save this file as main.tf

provider "azurerm" {
  features {}
}

# Variables for Subscription Creation
variable "billing_scope" {
  description = "The billing account or enrollment account ID."
  type        = string
}

variable "subscription_display_name" {
  description = "The display name for the new subscription."
  type        = string
}

# Resource: Azure Subscription
resource "azurerm_subscription" "new_subscription" {
  billing_scope          = var.billing_scope
  display_name           = var.subscription_display_name
}

# Output the Subscription ID
output "subscription_id" {
  description = "The ID of the newly created Azure Subscription."
  value       = azurerm_subscription.new_subscription.subscription_id
}

output "subscription_name" {
  description = "The display name of the newly created Azure Subscription."
  value       = azurerm_subscription.new_subscription.display_name
}
```

### Usage Example
1. Save the module in a directory called `azure_subscription`.
2. Create a `main.tf` file outside the module for invoking it:
   ```hcl
   module "create_subscription" {
     source                  = "./azure_subscription"
     billing_scope           = "YOUR_BILLING_ACCOUNT_ID"
     subscription_display_name = "NewSubscriptionName"
   }

   output "created_subscription_id" {
     value = module.create_subscription.subscription_id
   }

   output "created_subscription_name" {
     value = module.create_subscription.subscription_name
   }
   ```

3. Run the following Terraform commands:
   ```bash
   terraform init
   terraform plan
   terraform apply
   ```

4. Terraform will create the subscription and output its `ID` and `name`.

### Notes
- Replace `YOUR_BILLING_ACCOUNT_ID` with your actual Azure billing scope.
- Azure Subscription creation may require specific permissions. Make sure your service principal or Azure CLI credentials have adequate rights.

Let me know if you need more details or assistance!

# Azure-Policy-On-Resource-Group



To make the **Policy Definition for Tagging** a reusable Terraform module, you can structure your Terraform code into two parts: the module definition and the main script that uses the module. I'll guide you step by step to create the module and update the main script accordingly.

### Step 1: Create a Module for Policy Definition

First, you'll create a module that encapsulates the logic for creating a policy definition to enforce tagging. You'll store this module in a separate folder.

#### Folder Structure

```
/terraform/
    ├── main.tf
    ├── modules/
    │   └── tagging-policy/
    │       └── main.tf
    │       └── variables.tf
    │       └── outputs.tf
```

#### `main.tf` (Inside `modules/tagging-policy/`)

This file defines the logic for creating the policy definition and assigning it.

```hcl
# Create Policy Definition to enforce multiple tags
resource "azurerm_policy_definition" "tagging_policy" {
  name         = var.policy_name
  policy_type  = "Custom"
  mode         = "Indexed"
  display_name = var.display_name
  description  = var.description
  



  # The policy rule will check if any tag from the array does not exist, and if so, deny the resource creation
   policy_rule = <<POLICY_RULE
    {
    "if": {
      "not": {
        "field": "Tags",
        "in": "[parameters('tagmyKeys')]"
      }
    },
    "then": {
      "effect": "Deny"
    }
  }
POLICY_RULE

  metadata = jsonencode({
    "version" : "1.0.0",
    "category": "Tags"
  })
  parameters = <<PARAMETERS
    {
    "tagmyKeys": {
      "type": "Array",
      "metadata": {
        "description": "The list of allowed tags for resources.",
        "displayName": "Allowed tags",
        "strongType": "Tags"
      }
    }
  }
PARAMETERS
}

# Assign the policy to the specified scope
resource "azurerm_resource_group_policy_assignment" "tagging_policy_assignment" {
  name                 = var.assignment_name
  policy_definition_id = azurerm_policy_definition.tagging_policy.id
  resource_group_id    = var.scope
  display_name         = var.assignment_display_name
  description          = var.assignment_description

  # Since we are using multiple tags, there is no specific parameter needed for this part.
  # However, this can be adapted if we need to specify parameters per tag key.
  parameters = jsonencode({
    "tagmyKeys" : {
      "value" : var.tag_keys
    }
  })
}
```

#### `variables.tf` (Inside `modules/tagging-policy/`)

Define the input variables that will be passed to the module.

```hcl
# Define variables for the tagging policy module

variable "policy_name" {
  description = "The name of the policy"
  type        = string
}

variable "display_name" {
  description = "The display name of the policy"
  type        = string
}

variable "description" {
  description = "Description of the policy"
  type        = string
}

variable "tag_key" {
  description = "The key of the tag that must be enforced"
  type        = string
}

variable "assignment_name" {
  description = "Name of the policy assignment"
  type        = string
}

variable "scope" {
  description = "The scope at which the policy should be enforced"
  type        = string
}

variable "assignment_display_name" {
  description = "The display name of the policy assignment"
  type        = string
}

variable "assignment_description" {
  description = "Description of the policy assignment"
  type        = string
}
```

#### `outputs.tf` (Inside `modules/tagging-policy/`)

This file can provide outputs, such as the policy assignment ID, if needed.

```hcl
# Output the policy assignment ID
output "policy_assignment_id" {
  value = azurerm_policy_assignment.tagging_policy_assignment.id
}
```

### Step 2: Use the Module in the Main Script

Now that the module is ready, we’ll update the main Terraform script to use this module.

#### `main.tf` (At the root level)

```hcl

provider "azurerm" {
    
subscription_id = "4501a4d3-74c8-4703-9948-8c405a64daf0"
  features {}
}

# Create Resource Group
resource "azurerm_resource_group" "rg" {
  name     = "example-resource-group"
  location = "East US"
  
  tags = {
    environment = "production"
    owner       = "operations"
  }
}

# Call the tagging policy module, passing an array of tags
module "tagging_policy" {
  source                  = "./modules/tagging-policy"
  policy_name             = "require-tags"
  display_name            = "Require specific tags on all resources"
  description             = "This policy enforces that specific tags are provided on all resources."
  
  # Pass an array of tags
  tag_keys                = ["environment", "owner"]
  
  assignment_name         = "require-tags-assignment"
  scope                   = azurerm_resource_group.rg.id
  assignment_display_name = "Require multiple tags"
  assignment_description  = "This policy assignment ensures that all resources in the group have the specified tags."
}
```

### Step 3: Explanation

1. **Module Definition**:
   - The module defines both the policy and its assignment to a specified scope (in this case, the resource group). This makes it reusable for different tags and scopes.
   - `main.tf` defines the policy rule to check for the existence of a specified tag and denies creation if the tag is missing.
   
2. **Variables**:
   - In `variables.tf`, we define several input variables for flexibility, such as the tag key (`environment`), policy name, and the scope at which the policy is applied.

3. **Module Usage**:
   - The root-level `main.tf` script calls the module and passes the necessary variables like the tag key, policy name, display name, and the scope (resource group in this case).

### Step 4: How to Run

1. Navigate to the root folder (`terraform/`).
2. Initialize Terraform:
   ```bash
   terraform init
   ```
3. Run Terraform plan:
   ```bash
   terraform plan
   ```
4. Apply the configuration:
   ```bash
   terraform apply
   ```


To pass an array of `tag_key` values to enforce multiple required tags in the Azure Policy, we need to modify both the module and the policy definition. Terraform's `jsonencode` function can handle arrays, so we can adjust the policy to loop through each tag in the array and check for their existence.

Here’s how you can update the module and main script to support passing an array of tags.

### Step 1: Update the Module to Support an Array of `tag_key`

#### `main.tf` (Inside `modules/tagging-policy/`)

We modify the `policy_rule` to loop through the tags array and ensure that all tags exist. 

```hcl
# Create Policy Definition to enforce multiple tags
resource "azurerm_policy_definition" "tagging_policy" {
  name         = var.policy_name
  policy_type  = "Custom"
  mode         = "Indexed"
  display_name = var.display_name
  description  = var.description

  # The policy rule will check if any tag from the array does not exist, and if so, deny the resource creation
  policy_rule = jsonencode({
    "if" : {
      "anyOf" : [
        for tag in var.tag_keys : {
          "not" : {
            "field" : "tags[${tag}]",
            "exists" : true
          }
        }
      ]
    },
    "then" : {
      "effect" : "deny"
    }
  })

  metadata = jsonencode({
    "version" : "1.0.0",
    "category": "Tags"
  })
}

# Assign the policy to the specified scope
resource "azurerm_policy_assignment" "tagging_policy_assignment" {
  name                 = var.assignment_name
  policy_definition_id = azurerm_policy_definition.tagging_policy.id
  scope                = var.scope
  display_name         = var.assignment_display_name
  description          = var.assignment_description

  # Since we are using multiple tags, there is no specific parameter needed for this part.
  # However, this can be adapted if we need to specify parameters per tag key.
  parameters = jsonencode({
    "tagKeys" : {
      "value" : var.tag_keys
    }
  })
}
```

### Step 2: Update the `variables.tf` to Support an Array

We modify the `variables.tf` file to define `tag_keys` as a list of strings.

#### `variables.tf` (Inside `modules/tagging-policy/`)

```hcl
# Define variables for the tagging policy module

variable "policy_name" {
  description = "The name of the policy"
  type        = string
}

variable "display_name" {
  description = "The display name of the policy"
  type        = string
}

variable "description" {
  description = "Description of the policy"
  type        = string
}

variable "tag_keys" {
  description = "A list of tag keys that must be enforced"
  type        = list(string)
}

variable "assignment_name" {
  description = "Name of the policy assignment"
  type        = string
}

variable "scope" {
  description = "The scope at which the policy should be enforced"
  type        = string
}

variable "assignment_display_name" {
  description = "The display name of the policy assignment"
  type        = string
}

variable "assignment_description" {
  description = "Description of the policy assignment"
  type        = string
}
```

### Step 3: Use the Module and Pass an Array of Tags

Now in your main script, you can pass an array of tag keys.

#### `main.tf` (At the root level)

```hcl
provider "azurerm" {
  features {}
}

# Create Resource Group
resource "azurerm_resource_group" "rg" {
  name     = "example-resource-group"
  location = "East US"
  
  tags = {
    environment = "production"
    owner       = "operations"
  }
}

# Call the tagging policy module, passing an array of tags
module "tagging_policy" {
  source                  = "./modules/tagging-policy"
  policy_name             = "require-tags"
  display_name            = "Require specific tags on all resources"
  description             = "This policy enforces that specific tags are provided on all resources."
  
  # Pass an array of tags
  tag_keys                = ["environment", "owner"]
  
  assignment_name         = "require-tags-assignment"
  scope                   = azurerm_resource_group.rg.id
  assignment_display_name = "Require multiple tags"
  assignment_description  = "This policy assignment ensures that all resources in the group have the specified tags."
}
```

### Explanation:

1. **Policy Rule**: The updated policy rule checks each tag in the array (`tag_keys`). If any of the tags don't exist on the resource, it denies the creation of the resource.
   
2. **Array of Tags**: In the root-level script, you can pass an array of tags to the `tag_keys` parameter. For example: `["environment", "owner"]`.

3. **Looping in Policy Rule**: The `anyOf` clause is used to ensure that if any of the tags in the array is missing, the policy will be triggered and deny the resource creation.

### Step 4: How to Run

1. Navigate to the root folder (`terraform/`).
2. Initialize Terraform:
   ```bash
   terraform init
   ```
3. Run Terraform plan:
   ```bash
   terraform plan
   ```
4. Apply the configuration:
   ```bash
   terraform apply
   ```

This will enforce that resources in the resource group must have both the `environment` and `owner` tags (or any other tags you specify).
