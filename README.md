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
# Create Policy Definition to enforce tagging
resource "azurerm_policy_definition" "tagging_policy" {
  name         = var.policy_name
  policy_type  = "Custom"
  mode         = "Indexed"
  display_name = var.display_name
  description  = var.description

  policy_rule = jsonencode({
    "if" : {
      "not" : {
        "field" : "tags.${var.tag_key}",
        "exists" : true
      }
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

  parameters = jsonencode({
    "tagName" : {
      "value" : var.tag_key
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
  features {}
}

# Create Resource Group
resource "azurerm_resource_group" "rg" {
  name     = "example-resource-group"
  location = "East US"
  
  tags = {
    environment = "production"
  }
}

# Call the tagging policy module
module "tagging_policy" {
  source                  = "./modules/tagging-policy"
  policy_name             = "require-tags"
  display_name            = "Require a specific tag on all resources"
  description             = "This policy enforces that a 'environment' tag is provided on all resources."
  tag_key                 = "environment"
  assignment_name         = "require-tags-assignment"
  scope                   = azurerm_resource_group.rg.id
  assignment_display_name = "Require environment tag"
  assignment_description  = "This policy assignment ensures that all resources in the group have the 'environment' tag."
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

With this setup, you now have a reusable Terraform module for creating tagging policies, which can be applied to different resources and scopes by simply calling the module and passing different variables.
