# github-workflows-nbt-cloud-engineering

## terraform-azure.yml

A workflow to provide a basic Terraform Plan & Apply for Azure based deployments.

Assumptions:
- You are managing resources using the `azuread` or `azurerm` Terraform providers
- You are using the `azurerm` backend
- You have the required secrets populated in GitHub (can be automated via `github-repo-management`)
- You are not introducing breaking changes to the functionality.
- If you are introducing breaking changes, tag your commit message with `breaking:`



## Example usage:
```
name: My Workflow Instance

on:
  pull_request:
    branches:
      - main
  push:
    branches:
      - main

jobs:
  terraform:
    uses: racwa/github-workflows-cloud-engineering/.github/workflows/terraform-azure.yml@v1
    with:
      arm_subscription_id: 00000000-0000-0000-0000-000000000000
      tf_backend_container: mycontainer
      tf_backend_key: mycontext.tfstate
      tf_backend_resource_group: myrg
      tf_backend_storage_account: mybackendstorage 
      tf_backend_subscription_id: 00000000-0000-0000-0000-000000000000 
      tfsec_args: Args for TFSec --tfvars-file="full path from project root to var file"
      terraform_options: -input=false -var-file="mycontext.tfvars"
    secrets:
      AZURE_CLIENT_ID: ${{ secrets.AZURE_CLIENT_ID }}
      AZURE_CLIENT_SECRET: ${{ secrets.AZURE_CLIENT_SECRET }}
      AZURE_CREDENTIALS: ${{ secrets.AZURE_CREDENTIALS }}
      AZURE_TENANT_ID: ${{ secrets.AZURE_TENANT_ID }}
```


## Opt In Settings

### Compare Plans

This feature enables the Terraform plan that was approved in the PR to be compared to the plan created after the merge to main.

These two plans could be different due to other code changes or changes made outside of the code e.g. in the Azure Portal.  

The chance that the plans are different increases with the time lapsed between the plan being generated in the PR and the time when the merge and it subsequent workflow run takes place.

If the two plans are the same then apply stage is run after the plan stage without interruption.

**If the plans are different then an additional approval step is required** so we can be confident that we're applying changes that we want.

Add the below code to the "with:" section to enable this feature.

```
      compare_plan: true
```

Note that this feature relies on Github enironments.

The followng two environments are requied to enable the approval step:
- WorkflowApprovalNone
- WorkflowApprovalRequired

These can be deployed to your git repository via https://github.com/niyamtech/nbt-github-repo-management

Example Code:

```
environments:
  - name: WorkflowApprovalNone
    reviewers: []
  - name: WorkflowApprovalRequired
    reviewers:
      - Cloud Engineering
```


### Use saved plans

This feature ensures that the plan created in the plan stage is saved and used in the apply stage.

**This reduces work and time taken as the apply stage doesn't have to rerun the plan**.  

It also helps ensure that we are applying what we originally planned.

During the plan stage the created plan file is saved to the backend storage account.  Then it is downloaded and used during the apply stage.

Add the below code to the "with:" section to enable this feature.

```
      use_saved_plan: true
```

### Apply Empty Plan

This feature ensures that the Apply stage is not run if the Terraform plan says there are no changes to apply.

This reduces work and time taken as the apply stage doesn't have to run. 

Add the below code to the "with:" section to enable this feature.

```
      apply_empty_plan: true
```
