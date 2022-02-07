# Building an AutoApproval Flow in Azure Pipelines

Did you ever wanted to approve Azure Pipelines outside Azure DevOps? Perhaps you wanted the approval to take place inside Microsoft Teams? If this is the case, please keep reading. 

## Result

First of all the result of this story about an AutoApproval Flow in Azure Pipelines:

![result_autoapprovalflow](/.attachments/drawings/autoapprovalflow/result.png "This is our diagram and result for our AutoApprovalFlow")

## Quick set-up

How do we create this for our own? Follow my steps described below and leave a message if something is not working or missing:

Prerequisites:

1. Premiun license present for Power Automate, read [here](https://docs.microsoft.com/en-us/power-platform/admin/power-automate-licensing/faqs#who-needs-to-purchase-a-premium-license) more about how this can be obtained.
2. Some knowledge of Power Automate and [basic expressions](https://powerautomate.microsoft.com/nl-nl/blog/use-expressions-in-actions/).
3. Some knowledge of PowerShell.
4. Some knowledge of Azure DevOps and Azure Pipelines.

Installation:

1. At first we need to create our Flow in Power Automate, we will name it **AutoApprovalFlow**. To do so go to your Power Automate [environment](https://emea.flow.microsoft.com/manage/) and import this [file](/src/PowerAutomate/AutoApprovalFlow/AutoApprovalFlow_20220206094838.zip).
    - While importing the file above you need to update the name, Teams Connection and the Approval Connection. Look at below screenshot:
![autoapprovalflowimport](/.attachments/drawings/autoapprovalflow/import.png "This is our diagram and result for our AutoApprovalFlow")
2. After we imported our **AutoApprovalFlow** in Power Automate we need to edit the flow and:
    - Write down the HTTP Post URL:
![autoapprovalflowhttpposturl](/.attachments/drawings/autoapprovalflow/httpposturl.png "This is our diagram and result for our AutoApprovalFlow")
- Fill in the PAT in these three steps:
    - *Start getting the approval ID from the current build in Azure DevOps*
    - *Start patching the current build and cancelling the current build*
    - *Patch the current build so the approval request is handled*
![autoapprovalflowpat](/.attachments/drawings/autoapprovalflow/pat.png "This is our diagram and result for our AutoApprovalFlow")
3. Save the changes in our **AutoApprovalFlow** and make sure you have written down the HTTP Post Url in our previous steps.
4. In Azure DevOps we first need an Environment in Azure Pipelines. Lets [create](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops#create-an-environment) one named **AutoApprovalFlow**.
    - Go to ![autoapprovalflowdots](/.attachments/drawings/autoapprovalflow/dots.png "This is our diagram and result for our AutoApprovalFlow") and select Approval and Checks.
    - Select Approvals in the next screen ![autoapprovalflowapprovalcheck](/.attachments/drawings/autoapprovalflow/approvalcheck.png "This is our diagram and result for our AutoApprovalFlow") and add yourself for now. If you want to know more about approvals, click [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops&tabs=check-pass#approvals).
5. So we have our Power Automate Flow ready, written down the HTTP Post URL, created our Environment in Azure Pipelines and now we can create our Pipeline. Create a new Azure Pipeline called **AutoApproval** and use this [example](/src/Pipelines/AutoApprovalFlow/main.yml).
    - If you want to know how to create a Azure Pipeline, follow this course [here](https://docs.microsoft.com/nl-nl/learn/modules/create-a-build-pipeline/).
6. So we have our pipeline containing the YML from the [example](/src/Pipelines/AutoApprovalFlow/main.yml). Go and edit the pipeline and adjust the following code snippets in the YML to your situation:

```
$HTTPPOSTURL = "YOUR Power Automate URL HERE"
```

```
$autoApprovalObject | Add-Member -MemberType NoteProperty -Name "verificationOwner" -Value ("YOUR Verification Owner HERE")
```

7. After editing the YML in our pipeline we can start running the pipeline and test if this is working.