---
layout: post
title: "Azure DevOps Self-Hosted Agents with Azure Kubernetes Service"
date: 2022-08-09 17:00:00 +0200
categories:
  [
    azure,
    azure devops,
    azure kubernetes service,
    azure devops self hosted agents,
  ]
---

You have found the blog about Self-Hosted Agents in Azure DevOps. But we also explain something about Private Runners on GitHub, our main focus however is Azure DevOps. What can you read here?

- Explaining [why](#why-even-bother) this can be important to your organization or development team.
- Comparing some [scenarios](#why-even-bother) from a development, container and costs perspective.
- Showing you how to [configure](#azure-devops-self-hosted-agents) this for Azure Kubernetes Services.
- TOC
  {:toc}

# Why even bother?

So you must have heard about Microsoft Managed Agents, not? Read [here](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml) more about it.

But lets get to the why!? Because when you have something like Microsoft Managed Agents in Azure DevOps, why do we need Self-Hosted Agents at all (or in GitHub they are called Private Runners).

Lets compare these three (managed, self-hosted and private runners):

|                                                                | (Azure DevOps) Managed Agents                         | (Azure DevOps) Self-Hosted Agents                                                                                | (GitHub) Private Runners                                                                                                  |
| -------------------------------------------------------------- | ----------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Maintenance                                                    | None                                                  | ACI: none, AKS: cluster version upgrade, Container Apps: none                                                    | ACI: none, AKS: cluster version upgrade, Container Apps: none                                                             |
| Custom Image                                                   | No                                                    | Yes, with Azure Container Registry                                                                               | Yes, with Azure Container Registry                                                                                        |
| Requires installation of custom tooling, modules etc. each job | Yes                                                   | No                                                                                                               | No                                                                                                                        |
| VNET integration                                               | No                                                    | Yes                                                                                                              | Yes                                                                                                                       |
| Horizontal scaling (single to multiple agents/runners)         | Yes (max 20 in your entire organization)              | Yes (max 20 in your entire organization - ACI single instance, AKS no limit, Container Apps max 30 per instance) | Yes (max 180 / 500 depending on plan and license - ACI single instance, AKS no limit, Container Apps max 30 per instance) |
| Vertical scaling (more cores, RAM etc.)                        | No                                                    | Yes                                                                                                              | Yes                                                                                                                       |
| Costs (per parallel job)                                       | € 38,47 (1,800 minutes free with 1 free parallel job) | € 14,42 (first job is free)                                                                                      | Are free to use with GitHub Actions, but you are responsible for the cost of maintaining your runner machines             |
| Costs (average based on SKU B2ms per VM size)                  | Included in parallel job pricing                      | € 22,16                                                                                                          | € 22,16                                                                                                                   |

## So what can we conclude to the comparison table above?

That depending on your scenario Managed is by default the way to go. As soon as you have requirements to the machines then almost every time Managed is not your solution. So below I have a list of scenarios described with a bit more detail:

- **Maintenance**; so think of upgrading the AKS-cluster version.
- **Custom Image**; here we can decide if we need other software pre-installed on our machine. Check out the [Azure DevOps](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/hosted?view=azure-devops&tabs=yaml#software) and [GitHub](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#preinstalled-software) list of pre-installed software to match your requirements.
- **Requires installation of custom tooling, modules etc. each job**; this is applicable to the case you require additional tooling that needs to be installed (besides the pre-installed software). In the scenario of Managed it will be installed each time, which can take a lot of seconds/minutes or more to complete your Pipeline/GitHub Action. In the other scenarios we can pre-install this and reduce the total run time of our Pipeline/GitHub Action.
- **VNET integration**; so in the case you need to isolate the machines or have a requirement to access endpoints within a specific network (VNET) then you can achieve this for not Managed scenarios. This also depends on the type of traffic and if you need to access on-premises.
- **Horizontal scaling**; if you have a single job, each scenario can be ran. If you require multiple jobs to be ran each time then check the requirements/limitations [above](#why-even-bother).
- **Vertical scaling**; so we assume most development teams don't require that much processing power (CPU) or RAM. But in the case you do, we need to switch to the not Managed scenarios. This can be applicable for multi-core processes or ETL workloads with specific hardware requirements. The Azure DevOps Managed Agent and GitHub Hosted Runner hardware consists of 2 core CPU, 7 GB of RAM, and 14 GB of SSD disk space. For GitHub there are Larger runners available with more capacity, check it [here](https://docs.github.com/en/actions/using-github-hosted-runners/using-larger-runners#machine-specs-for-larger-runners).
- **Costs**; somehow important right? Together with the scenarios above, total runtime of your jobs per day/month we can calculate the total costs and compare them with each other. Keep in mind that in Azure we can also apply dev/test subscriptions and reserved instances to reduce costs even more.

## But the pricing of managed versus other solutions, how does that end up?

That is easy to say, so lets look at this table where we check the licenses for three scenarios:

|                             | (Azure DevOps) Managed Agents | (Azure DevOps) Self-Hosted Agents | (GitHub) Private Runners |
| --------------------------- | ----------------------------- | --------------------------------- | ------------------------ |
| 1 Agent / Private Runner    | € 38,47                       | € 0,00                            | € 0,00                   |
| 2 Agents / Private Runners  | € 76,94                       | € 14,42                           | € 0,00                   |
| 10 Agents / Private Runners | € 384,70                      | € 129.83                          | € 0,00                   |

Now lets calculate our total costs in these scenarios where we add the costs for running these machines (based on 6 hours each day):

- Our AKS-cluster costs € 17,17 per VM running 6 hours a day. This is based on the Azure Pricing Calculator and can differ.
- The used SKU B2ms can be compared to the Azure DevOps and GitHub Managed Agent/Runner hardware containing 2 core CPU, 7 GB of RAM, and 14 GB of SSD disk space. For GitHub there are Larger runners available with more capacity, check it [here](https://docs.github.com/en/actions/using-github-hosted-runners/using-larger-runners#machine-specs-for-larger-runners).

|                             | (Azure DevOps) Managed Agents | (Azure DevOps) Self-Hosted Agents | (GitHub) Private Runners |
| --------------------------- | ----------------------------- | --------------------------------- | ------------------------ |
| 1 Agent / Private Runner    | € 38,47                       | € 17,17                           | € 17,17                  |
| 2 Agents / Private Runners  | € 76,94                       | € 48,76                           | € 34,34                  |
| 10 Agents / Private Runners | € 384,70                      | € 301,53                          | € 171,70                 |

## Should I move to my own Self-Hosted Agents?

Depending on your scenario and requirements. I will not say you must do this!

However my personal preference is to install my own Self-Hosted Agents each time, no matter if it is Azure DevOps or GitHub. There are just too many benefits from having your own Self-Hosted Agents or Private Runners.

Also consider the following:

- Management of these Self-Hosted Agents
- Alerting and monitoring

If you have any questions about this please let me know!?

# Installation Guide - Azure DevOps Self-Hosted Agents

This is a complete guide how to build your own Azure DevOps Self-Hosted Agents. This is partially done local with Docker Desktop, in Azure with Azure Kubernetes Service (**AKS**), Azure Container Registry (ACR) and we can even apply auto-scaling when required to your scenario. All of this written in code and deployed from Azure CLI (also configurable with CI/CD in Azure Pipelines or GitHub Actions).

Pre-requisites before starting:

- A Azure DevOps Project with full access (Project Administrators group), this is the place where our Agent Pool will be configured.
- A Azure Tenant with at least one (active) Azure Subscription where you want to deploy your AKS and ACR to.
- The RBAC-role Owner assigned to your Azure AD Account on the (active) Azure Subscription. This is required because we need to register some Resource Providers.
- [Visual Studio Code](https://code.visualstudio.com/) (or another code editor).
- [PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.2).
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).

## Part 1. Build, configure and test your image with Docker Desktop

You can build several images, the hardest one I found to configure was Windows. So that is also the one we are focusing on here. Before we start we need to have the following information:

1.  Azure DevOps PAT token, how to create one go [here](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows#create-a-pat).
2.  Azure DevOps URL, for example: https://dev.azure.com/**\<organization name\>\*\*
3.  Azure DevOps Pool Name, create one [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/pools-queues?view=azure-devops&tabs=yaml%2Cbrowser).
4.  Azure DevOps Pool ID, retrieve this by running the script below (replace the current variables with your values):

        $SYSTEM_ACCESSTOKEN = "<Azure DevOps PAT token>"
        $Headers = @{Authorization = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($SYSTEM_ACCESSTOKEN)")) }
        $URI = "https://dev.azure.com/<organization name>/_apis/distributedtask/pools?api-version=6.0"
        $Result = Invoke-WebRequest -Uri $URI -Method get -Headers $Headers -UseBasicParsing
        ($Result.Content | ConvertFrom-Json).value | Select-Object name, id

Do you want to build this with Linux? No problem, go to the Dockerfile [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#create-and-build-the-dockerfile-1) and use it in the next steps:

1. Download [Docker Desktop](https://docs.docker.com/get-docker/) for your Operating System and install Docker Desktop. Since I am using Windows I have selected Windows.

   > If there is any issue with installing or using Docker Desktop in your Operating System then I can help you out but I do not have any troubleshooting tips on this page.

2. After the installation choose a location where you want to save the files we are about to make. If you are coding inside a repository just create a new branch and a folder of your choosing. I created the folder: **C:\dockeragent**
3. Inside this folder we save the DockerFile which can be found [here](https://github.com/buys-stash/in-a-box/blob/main/azdevops-selfhostedagents/dockerimages/windows/Dockerfile).
4. Also inside the same folder we save a file called start.ps1 which can be found [here](https://github.com/buys-stash/in-a-box/blob/main/azdevops-selfhostedagents/dockerimages/windows/start.ps1).
5. Switch with Docker Desktop to Windows container modus by right clicking on the running Docker Desktop icon (in Windows this is on your taskbar at the right side).

![switch_windows_containers](/.attachments/switchtowindowscontainers.jpeg)

6.  Now start a new PowerShell-window or use the terminal inside Visual Studio Code.
7.  Navigate to the folder you saved the previous created files.

        cd c:\dockeragent

8.  Run this command to build a new image from the created Dockerfile with the **name** dockeragent and the **tag** latest.

        docker build -t dockeragent:latest .

9.  The image is build, now we can run and test it.

With all the required information and all the steps we did we can run and see if our image can be build and our container can register as a Azure DevOps Self-Hosted Agent.  
Now start a new PowerShell-window or use the terminal inside Visual Studio Code and run the code below (replace the values with your values):

        $AZP_URL = https://dev.azure.com/**<organization name>**
        $AZP_TOKEN = "<Azure DevOps PAT token>"
        $AZP_POOL = "<Azure DevOps Agent Pool Name>"
        docker run -e AZP_URL=$AZP_URL -e AZP_TOKEN=$AZP_TOKEN -e AZP_POOL=$AZP_POOL -e AZP_AGENT_NAME=mydockeragent dockeragent:latest

When the container is up and running go to your Azure DevOps organisation and navigate to your Agent Pool to see if **mydockeragent** is online (this can take a few minutes so be patient).

Related documentation: [Microsoft Docs - Run a self-hosted agent in Docker](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops)

## Part 2. Create your Azure Container Registry

So we have an image created and tested inside Docker Desktop which we can upload to an Azure Container Registry. This is required because AKS can use this image later on.  
Follow these steps:

1.  Start a new PowerShell-window or use the terminal inside Visual Studio Code.
2.  Login to Azure with the Azure CLI command:

        az login

3.  Select your subscription:

        az account set --subscription "<subscription id>"

4.  Create the resource group where we are deploying our container registry (and AKS cluster later on):

        az group create --name "rg-selfhostedagents" --location westeurope

5.  Inside the resource group deploy the Azure Container Registry:

        az acr create --resource-group "rg-selfhostedagents" --name "cr-dockerimages" --sku Basic

Off course you can rename each value to your own value.

## Part 3. Upload your image to Azure Container Registry

Now we have a Azure Container Registry where our image can be pushed to. In order to do so, follow these steps:

1.  Start a new PowerShell-window or use the terminal inside Visual Studio Code.
2.  Make sure you are logged in Azure with the Azure CLI (follow Part 2. Step 1 till 3 how to do this) and have selected the correct Azure Subscription where your Azure Container Registry is deployed in.
3.  First we login to the Azure Container Registry.

        az acr login --name \<acrname\>

4.  Before we can push our image we need to give it the same name as our Azure Container Registry (this is required to push it to ACR from your local environment). In your current terminal go to the folder where the dockerfile is in and run the next command.

        cd c:\dockeragent
        docker build -t \<acrname\>.azurecr.io/dockeragent:latest .

5.  After the build is completed we can push it to our Azure Container Registry.

        docker push \<acrname\>.azurecr.io/dockeragent:latest

After this is done we successfully pushed the image to the Azure Container Registry, ready for our next step.

## Part 4. Build your AKS Cluster

We have a Azure Container Registry with an image tested running as a Azure DevOps Self-Hosted agent. Now we are going to create a AKS Cluster for this setup (based on Windows Server 2019):

1.  Start a new PowerShell-window or use the terminal inside Visual Studio Code.
2.  Make sure you are logged in Azure with the Azure CLI (follow Part 2. Step 1 till 3 how to do this) and have selected the correct Azure Subscription where you want to deploy the AKS Cluster in.
3.  Enter the Windows Username used to login to the nodes on your AKS Cluster.

        $WINDOWS_USERNAME = "WINDOWS USERNAME"

4.  For the next part we are going to create the AKS Cluster with a single node:

        az aks create `
        --resource-group "rg-selfhostedagents" `
        --name "aks-selfhostedagents" `
        --node-count 1 `
        --enable-addons monitoring `
        --generate-ssh-keys `
        --windows-admin-username $WINDOWS_USERNAME `
        --vm-set-type VirtualMachineScaleSets `
        --network-plugin azure

5.  After our deployment we are going to configure a node pool to support our Windows Image (created in the first Part of this guide):

        az aks nodepool add \
        --resource-group "rg-selfhostedagents" \
        --cluster-name "aks-selfhostedagents" \
        --os-type Windows \
        --name azhosts \
        --node-count 1

When the AKS Cluster and node pool are deployed you can continue to the next part of this guide.

## Part 5. Configure your AKS Cluster to deploy Azure DevOps Self-Hosted Agents

We have a Azure Container Registry (ACR) and a configured AKS Cluster, now we only need to configure our AKS Cluster to deploy Azure DevOps Self-Hosted Agents. Lets follow these steps:

1.  Go to the folder where your dockeragent images are, in this guide that would be:

        cd c:\dockeragent

2.  Lets create a subfolder called kubeconfig.

        mkdir c:\dockeragent\kubeconfig

3.  Inside this folder we will copy all the files in this [folder](https://github.com/buys-stash/in-a-box/tree/main/azdevops-selfhostedagents/kubernetes) to the subfolder we just created.
4.  After you copied all the required files we are going to login to our AKS Cluster (make sure you are logged in with the Azure CLI and selected the (active) Azure Subscription where you AKS Cluster is deployed in).

        az aks get-credentials --resource-group "rg-selfhostedagents" --name "aks-selfhostedagents"

5.  So we are in the subfolder kubeconfig and we are logged into our AKS Cluster. Lets verify this by checking our recently deployed node pool:

        kubectl get nodes

6.  If you verified that our nodepool **azdevops** is visible then you can continue, else check if you followed the previous steps correctly!
7.  First we need authentication set-up between our AKS Cluster and the Azure Container Registry so AKS can pull the image we created. Apply the following commands where we replace the NAME OF ACR with your ACR name. After that the commands will retrieve the credentials for the Azure Container Registry and use the first password available to add the required access in our AKS Cluster.

        $ACR_NAME = "NAME OF ACR"
        $REGISTRY_URL = "$ACR_NAME.azureacr.io"
        $ACR_CREDENTIALS = az acr credential show -n $ACR_NAME | ConvertFrom-Json
        $ACR_USERNAME = $ACR_CREDENTIALS.username

        $ACR_PASSWORD = $ACR_CREDENTIALS.passwords[0].value
        **OR**
        $ACR_PASSWORD = $ACR_CREDENTIALS.passwords.value

        kubectl create secret docker-registry acr-secret --docker-server=$REGISTRY_URL --docker-username=$ACR_USERNAME --docker-password=$ACR_PASSWORD

        az acr update -n $ACR_NAME --admin-enabled true

8.  Before we apply all the recently copied files we need to adjust some files, lets open the Secret.yml and replace the values mentioned between <> and save the file:

        apiVersion: v1
        kind: Secret
        metadata:
          name: azdevops
          namespace: azdevops-agents
        type: Opaque
        stringData:
          AZP_POOL: <your Azure DevOps Pool name, look at part 1 for more information>
          AZP_TOKEN: <your Azure DevOps Personal Access Token, look at part 1 for more information>
          AZP_URL: <your Azure DevOps Organization URL, look at part 1 for more information>

9.  Now lets open the ReplicationController.yml and replace the values mentioned between <> (in this file it is only the image URL pointing to your ACR) and save the file:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: azhosts
          namespace: azdevops-agents
          labels:
            app: azdevops-agent
        spec:
          replicas: 1 #here is the configuration for the actual agent always running
          selector:
            matchLabels:
              app: azdevops-agent
          template:
            metadata:
              labels:
                app: azdevops-agent
            spec:
              containers:
              - name: azdevopsagentcreation
                image: \<acrname\>.azurecr.io/dockeragent:latest
                env:
                  - name: AZP_URL
                    valueFrom:
                      secretKeyRef:
                        name: azdevops
                        key: AZP_URL
                  - name: AZP_TOKEN
                    valueFrom:
                      secretKeyRef:
                        name: azdevops
                        key: AZP_TOKEN
                  - name: AZP_POOL
                    valueFrom:
                      secretKeyRef:
                        name: azdevops
                        key: AZP_POOL
                volumeMounts:
                - mountPath: /var/run/docker.sock
                  name: docker-volume
              imagePullSecrets:
                - name: acr-secret
              volumes:
              - name: docker-volume
                hostPath:
                  path: /var/run/docker.sock

10. Lets apply all the files (use full filepaths if you are not in the directory where these files are stored inside your current terminal session):

        kubectl apply -f Namespace.yml
        kubectl apply -f Secret.yml
        kubectl apply -f ReplicationController.yml

11. Lets verify our current configuration by checking if the namespace **azdevops-agents** is created:

        kubectl get namespaces

12. Lets verify if the secret azdevops is created, but before we do that lets switch the current context to our newly created namespace:

        kubectl config set-context --current --namespace=azdevops-agents
        kubectl get secret

13. Finally lets verify if the deployment azhosts is created:

        kubectl get deployment

14. After the configuration your AKS Cluster should be up and running, deploying Azure DevOps Self-Hosted Agents. To check this:

        kubectl get pods

15. If you see a pod running with the prefix name azdevops-... then go to the Azure DevOps Agent Pool in the Azure DevOps portal and verify an agent is online.

If this is not working please contact me or go to the troubleshooting Part 7.2 of this guide.

## Part 6. Configure KEDA auto-scaling for the Azure DevOps Self-Hosted Agents inside AKS Cluster

So you have an AKS Cluster and Azure Container Registry up and running but also did some configuration to link these two with the proper authentication. Now you need auto-scaling so lets configure this with the following steps:

1.  Go to the folder where your dockeragent images are and navigate to the subfolder kubeconfig:

        cd c:\dockeragent\kubeconfig

2.  Follow Step 4 in Part 1 to get the Azure DevOps Pool ID of the Pool you want to deploy Azure DevOps Self-Hosted Agents in.
3.  Lets prepare our AKS Cluster to configure KEDA (which is the mechanisme to enable auto-scaling of pods). Make sure you are logged into Azure, selected the right Azure Subscription and are logged into your AKS Cluster. Then run these commands:

        az feature register --name AKS-KedaPreview --namespace Microsoft.ContainerService
        az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKS-KedaPreview')].{Name:name,State:properties.state}"
        az provider register --namespace Microsoft.ContainerService
        az extension add --upgrade --name aks-preview
        az aks update `
        --resource-group "rg-selfhostedagents" `
        --name "aks-selfhostedagents" `
        --enable-keda

4.  Lets check if KEDA is enabled on our AKS Cluster:

        az aks show -g "rg-selfhostedagents" --name "aks-selfhostedagents" --query "workloadAutoScalerProfile.keda.enabled"

5.  If KEDA is enabled then we can apply the scaledObject which contains the scaling configuration for our AKS Cluster. Replace the poolID with the ID you retrieved in step 2:

        apiVersion: keda.sh/v1alpha1
        kind: TriggerAuthentication
        metadata:
          name: pipeline-trigger-auth
        spec:
          secretTargetRef:
            - parameter: personalAccessToken
              name: azdevops
              key: AZP_TOKEN
        ---
        apiVersion: keda.sh/v1alpha1
        kind: ScaledObject
        metadata:
          name: azure-pipelines-scaledobject
        spec:
          scaleTargetRef:
            name: azdevops
          minReplicaCount: 1
          maxReplicaCount: 5
          triggers:
          - type: azure-pipelines
            metadata:
              poolID: "<ID OF YOUR Azure DevOps Pool>"
              organizationURLFromEnv: "AZP_URL"
            authenticationRef:
            name: pipeline-trigger-auth

6.  Apply the configuration:

        kubectl apply -f ScaledObject.yml

7.  Retrieve the ScaledObject and check if Ready is set to True and Active is set to False:

        kubectl get ScaledObject

8.  To test this you can configure your pipelines to use the Azure DevOps Agent Pool we configured for auto-scaling, look at the snippet below containing a piece of a YAML pipeline:

        jobs:
        - job: "Example of Jobname"
          timeoutInMinutes: 0
          pool: 'YOUR AGENT POOL'

9.  After adding the correct pool to the pipeline, run the pipeline multiple times to see if auto-scaling is working (it requires multiple jobs and some patience to see it working correctly on a Windows image). Another option you can try is this loop to start multiple jobs and trigger auto-scaling (replace the values with your own values):

        $URI = "https://dev.azure.com/<organisation>/<project>/_apis/build/builds?api-version=6.0";"
        $BuildDefinitionID = '<your definition id>'
        $Body = New-Object -TypeName PSObject -Property @{}

        $definition = New-Object -TypeName PSObject -Property @{
            id = $BuildDefinitionID
        }

        $Body | Add-Member -MemberType NoteProperty -Name "definition" -Value ($definition)

        $i=1

        for (;$i -le 20;$i++){
          Invoke-WebRequest -Uri $URI -Method get -Headers $Headers -UseBasicParsing
        }

# Can I do this in a CI/CD Pipeline or GitHub Action?

Off course, I have some files ready to get you up and running which can be located [here]().
