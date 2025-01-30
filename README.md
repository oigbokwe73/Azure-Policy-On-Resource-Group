
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
