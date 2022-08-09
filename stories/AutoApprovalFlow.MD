# Building an AutoApproval Flow in Azure Pipelines

Did you ever wanted to approve Azure Pipelines outside Azure DevOps? Perhaps you wanted the approval to take place inside Microsoft Teams? If this is the case, please keep reading. 

## Result

First of all the result of this story about an AutoApproval Flow in Azure Pipelines:

![result_autoapprovalflow](/.attachments/drawings/autoapprovalflow/result.png "This is our diagram and result for our AutoApprovalFlow")

## Quick set-up (time to complete: 15-30 minutes)

How do we create this for our own? Follow my steps described below and leave a message if something is not working or missing:

Prerequisites (some are not required since they are explaind inside this demo):

1. Premium license present for Power Automate, read [here](https://docs.microsoft.com/en-us/power-platform/admin/power-automate-licensing/faqs#who-needs-to-purchase-a-premium-license) more about how this can be obtained.
2. Some knowledge of Power Automate and [basic expressions](https://powerautomate.microsoft.com/nl-nl/blog/use-expressions-in-actions/).
3. Some knowledge of PowerShell regarding sending a JSON payload to an endpoint like Power Automate (optional).
4. Some knowledge of Azure DevOps and Azure Pipelines regarding YML, Environments and using Pipelines (optional).
5. Azure DevOps environment with a project capable of running Azure Pipelines.
6. We have the Azure DevOps organization name written down. This will be used later on.
7. We have the Azure DevOps project name written down. This will be used later on.
8. Some [connectors](https://emea.flow.microsoft.com/en-us/connectors/) which you can create before importing the flow:
    - Microsoft Teams
    - Approvals
    - Azure DevOps

Installation:

1. At first we need to create our Flow in Power Automate, we will name it **AutoApprovalFlow**. To do so go to your Power Automate [environment](https://emea.flow.microsoft.com/manage/) and import this [file](/src/PowerAutomate/AutoApprovalFlow/AutoApprovalFlow_20220218055854.zip).
    - While importing the file above you need to update the name, Teams Connection, Approval Connection and the Azure DevOps Connection. Look at below screenshot*:
![autoapprovalflowimport](/.attachments/drawings/autoapprovalflow/import.png "This is our diagram and result for our AutoApprovalFlow")
    - **Name**, at the Import Setup select Create as new with the name you preferr to use. For this demo go for AutoApproval.
    - **Teams Connection**, this is the connection required to communicate with Teams. If none exists select Create as new.
    - **Approval Connection**, this is the connection required to send the approvals to Teams. If none exists select Create as new.
    - **Azure DevOps Connection**, this is the connection required to communicate with the REST API of Azure DevOps. If none exists select Create as new.
2. After we imported our **AutoApprovalFlow** in Power Automate we need to edit the flow and:
    - Write down the HTTP Post URI from this step (first step in our flow):
![autoapprovalflowhttpposturl](/.attachments/drawings/autoapprovalflow/httpposturl.png "This is our diagram and result for our AutoApprovalFlow")
- Change the following details to your Azure DevOps **ENTER YOUR ORGANIZATION HERE** and **ENTER YOUR PROJECTNAMEHERE** in these three steps inside our imported flow (below you find the name of these three steps and a screenshot):
    - *Get current build details to retrieve ApprovalID*
    - *Cancel current build in Azure DevOps*
    - *Patch the current build so the approval request is handled*
![autoapprovalflowpat](/.attachments/drawings/autoapprovalflow/devopshttp.PNG "This is our diagram and result for our AutoApprovalFlow")
3. Save the changes in our **AutoApprovalFlow** and make sure you have written down the HTTP Post URI from our previous step.
4. In Azure DevOps we first need an Environment in Azure Pipelines. Lets [create](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/environments?view=azure-devops#create-an-environment) one named **AutoApprovalFlow** containing **Resource** none.
    - Go to ![autoapprovalflowdots](/.attachments/drawings/autoapprovalflow/dots.png "This is our diagram and result for our AutoApprovalFlow") and select Approval and Checks.
    - Select Approvals in the next screen ![autoapprovalflowapprovalcheck](/.attachments/drawings/autoapprovalflow/approvalcheck.png "This is our diagram and result for our AutoApprovalFlow") and add yourself for now. If you want to know more about approvals, click [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops&tabs=check-pass#approvals).
5. So we have our Power Automate Flow ready, written down the HTTP Post URI, created our Environment in Azure Pipelines and now we can create our Pipeline. Create a new Azure Pipeline called **AutoApproval** and use this [example](/src/Pipelines/AutoApprovalFlow/main.yml).
    - If you want to know how to create a Azure Pipeline, follow this course [here](https://docs.microsoft.com/nl-nl/learn/modules/create-a-build-pipeline/).
6. So we have our pipeline containing the YML from the [example](/src/Pipelines/AutoApprovalFlow/main.yml). Go and edit the pipeline and adjust the following code snippets in the YML to your situation:
    - **YOUR HTTP POST URI**; this is the HTTP Post URI written down from step 2.
    - **youraccount@approvalflow.nl**; this is the user (e-mail) used in our Power Automate flow to send the Approval to. This can be multiple users by seperating them with a comma delimiter.

```
$HTTPPOSTURI = "YOUR HTTP POST URI"
```

```
$autoApprovalObject | Add-Member -MemberType NoteProperty -Name "verificationOwner" -Value ("youraccount@approvalflow.nl")
```

7. After editing the YML in our pipeline we can start running the pipeline and test if this is working.
8. After the Pre stage it could be possible that permission needs to be granted so our pipeline can access our early created Environment. Give it the required permission.
9. After the Pre stage you should receive an approval request inside Teams (check your notifications in the side menu): ![autoapprovalflowapprovalcheck](/.attachments/drawings/autoapprovalflow/teamsresult.PNG)
10. You may approve this request to complete our flow and our pipeline waiting for approval in Azure DevOps.
11. Our pipeline will resume and you will receive a notification in Teams: ![autoapprovalflowapprovalcheck](/.attachments/drawings/autoapprovalflow/teamsresultapproved.PNG)

You have completed the AutoApprovalFlow setup combining Azure Pipelines with Power Automate, Microsoft Teams and using YML, JSON and PowerShell.

Off course this is a simple example of all the possibilities you can achieve with the above, some examples:

- Extend this flow with a Wait untill so that when an approval request is approved the flow will only continue after a certain date and time.
- Extend this flow with multiple messages to inform people during the flow so they are aware of the current state of the flow.
- And many more you can think of since this is just the beginning of something great...
