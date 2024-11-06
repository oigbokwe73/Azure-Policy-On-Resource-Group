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
