---
layout: post
title:  "Azure DevOps Self-Hosted Agents with Azure Kubernetes Service"
date:   2022-08-09 17:00:00 +0200
categories: jekyll update
---

Looking for a way to deploy Azure DevOps Self-Hosted Agents and run them inside Azure Kubernetes Service (**AKS**)? Do you want your Linux distribution or Windows operating system loaded from Azure Container Registry? Or do you even need auto-scaling for your agents no matter it is a Linux distribution of Windows operating system?

**STOP**! This is the place where you will find 'how' to do this. Just follow each part of the table of contents or pick the chapter related to your situation or question.

* TOC
{:toc}

# Azure DevOps Self-Hosted Agents

This blog is a complete guide how to build your own Azure DevOps Self-Hosted Agents. This is partially done local with Docker Desktop, in Azure with Azure Kubernetes Service  (**AKS**), Azure Container Registry (ACR) and we can even apply auto-scaling when required to your scenario. All of this written in code and deployed from Azure CLI (also configurable with CI/CD in Azure Pipelines or GitHub Actions).

Pre-requisites before starting:
- A Azure DevOps Project with full access (Project Administrators group), this is the place where our Agent Pool will be configured.  
- A Azure Tenant with at least one (active) Azure Subscription where you want to deploy your AKS and ACR to.  
- The RBAC-role Owner assigned to your Azure AD Account on the (active) Azure Subscription. This is required because we need to register some Resource Providers.  
- [Visual Studio Code](https://code.visualstudio.com/) (or another code editor).  
- [PowerShell](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell?view=powershell-7.2).  
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli).  
  
## Part 1. Build, configure and test your image with Docker Desktop

You can build several images, the hardest one I found to configure was Windows. So that is also the one we are focusing on here. Before we start we need to have the following information:

1. Azure DevOps PAT token, how to create one go [here](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=Windows#create-a-pat).
2. Azure DevOps URL, for example: https://dev.azure.com/**\<organization name\>**
3. Azure DevOps Pool Name, create one [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/pools-queues?view=azure-devops&tabs=yaml%2Cbrowser).
4. Azure DevOps Pool ID, retrieve this by running the script below (replace the current variables with your values):

        $SYSTEM_ACCESSTOKEN = "<Azure DevOps PAT token>"  
        $Headers = @{Authorization = 'Basic ' + [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$($SYSTEM_ACCESSTOKEN)")) }  
        $URI = "https://dev.azure.com/<organization name>/_apis/distributedtask/pools?api-version=6.0"  
        $Result = Invoke-WebRequest -Uri $URI -Method get -Headers $Headers -UseBasicParsing  
        ($Result.Content | ConvertFrom-Json).value | Select-Object name, id  

Do you want to build this with Linux? No problem, go to the Dockerfile [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#create-and-build-the-dockerfile-1) and use it in the next steps:

1. Download [Docker Desktop](https://docs.docker.com/get-docker/) for your Operating System and install Docker Desktop. Since I am using Windows I have selected Windows.
  > If there is any issue with installing or using Docker Desktop in your Operating System then I can  help you out but I do not have any troubleshooting tips on this page.
  
2. After the installation choose a location where you want to save the files we are about to make. If you are coding inside a repository just create a new branch and a folder of your choosing. I created the folder: **C:\dockeragent**
3. Inside this folder we save the DockerFile which can be found [here](https://github.com/buys-stash/in-a-box/blob/main/azdevops-selfhostedagents/dockerimages/windows/Dockerfile).
4. Also inside the same folder we save a file called start.ps1 which can be found [here](https://github.com/buys-stash/in-a-box/blob/main/azdevops-selfhostedagents/dockerimages/windows/start.ps1).
5. Now start a new PowerShell-window or use the terminal inside Visual Studio Code.
6. Navigate to the folder you saved the previous created files.  

        cd c:\dockeragent  
  
7. Run this command to build a new image from the created Dockerfile with the **name** dockeragent and the **tag** latest.  

        docker build -t dockeragent:latest .  

8. The image is build, now we can run and test it.  

With all the required information and all the steps we did we can run and see if our image can be build and our container can register as a Azure DevOps Self-Hosted Agent.  
Now start a new PowerShell-window or use the terminal inside Visual Studio Code and run the code below (replace the values with your values):  

        $AZP_URL = https://dev.azure.com/**\<organization name\>**
        $AZP_TOKEN = "<Azure DevOps PAT token>"
        $AZP_POOL = "<Azure DevOps Agent Pool Name>"
        docker run -e AZP_URL=$AZP_URL -e AZP_TOKEN=$AZP_TOKEN -e AZP_POOL=$AZP_POOL -e AZP_AGENT_NAME=mydockeragent dockeragent:latest

When the container is up and running go to your Azure DevOps organisation and navigate to your Agent Pool to see if **mydockeragent** is online (this can take a few minutes so be patient).

Related documentation: [Microsoft Docs - Run a self-hosted agent in Docker](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops)

## Part 2. Create your Azure Container Registry

So we have an image created and tested inside Docker Desktop which we can upload to an Azure Container Registry. This is required because AKS can use this image later on.  
Follow these steps:

1. Start a new PowerShell-window or use the terminal inside Visual Studio Code.
2. Login to Azure with the Azure CLI command:

        az login  

3. Select your subscription:

        az account set --subscription "<subscription id>"  

4. Create the resource group where we are deploying our container registry (and AKS cluster later on):

        az group create --name "rg-selfhostedagents" --location westeurope 

5. Inside the resource group deploy the Azure Container Registry:

        az acr create --resource-group "rg-selfhostedagents" --name "cr-dockerimages" --sku Basic  

Off course you can rename each value to your own value.

## Part 3. Upload your image to Azure Container Registry

Now we have a Azure Container Registry where our image can be pushed to. In order to do so, follow these steps:

1. Start a new PowerShell-window or use the terminal inside Visual Studio Code.
2. Make sure you are logged in Azure with the Azure CLI (follow Part 2. Step 1 till 3 how to do this) and have selected the correct Azure Subscription where your Azure Container Registry is deployed in.
3. First we login to the Azure Container Registry.

        az acr login --name \<acrname\>  

4. Before we can push our image we need to give it the same name as our Azure Container Registry (this is required to push it to ACR from your local environment). In your current terminal go to the folder where the dockerfile is in and run the next command.

        cd c:\dockeragent
        docker build -t \<acrname\>.azurecr.io/dockeragent:latest .  

6. After the build is completed we can push it to our Azure Container Registry.

        docker push \<acrname\>.azurecr.io/dockeragent:latest

After this is done we successfully pushed the image to the Azure Container Registry, ready for our next step.

## Part 4. Build your AKS Cluster

We have a Azure Container Registry with an image tested running as a Azure DevOps Self-Hosted agent. Now we are going to create a AKS Cluster for this setup (based on Windows Server 2019):

1. Start a new PowerShell-window or use the terminal inside Visual Studio Code.
2. Make sure you are logged in Azure with the Azure CLI (follow Part 2. Step 1 till 3 how to do this) and have selected the correct Azure Subscription where you want to deploy the AKS Cluster in.
3. Enter the Windows Username used to login to the nodes on your AKS Cluster.

        $WINDOWS_USERNAME = "WINDOWS USERNAME"

4. For the next part we are going to create the AKS Cluster with a single node:

        az aks create `
        --resource-group "rg-selfhostedagents" `
        --name "aks-selfhostedagents" `
        --node-count 1 `
        --enable-addons monitoring `
        --generate-ssh-keys `
        --windows-admin-username $WINDOWS_USERNAME `
        --vm-set-type VirtualMachineScaleSets `
        --network-plugin azure

5. After our deployment we are going to configure a node pool to support our Windows Image (created in the first Part of this guide):

        az aks nodepool add \
        --resource-group "rg-selfhostedagents" \
        --cluster-name "aks-selfhostedagents" \
        --os-type Windows \
        --name azdevops \
        --node-count 1

When the AKS Cluster and node pool are deployed you can continue to the next part of this guide.

## Part 5. Configure your AKS Cluster to deploy Azure DevOps Self-Hosted Agents

We have a Azure Container Registry (ACR) and a configured AKS Cluster, now we only need to configure our AKS Cluster to deploy Azure DevOps Self-Hosted Agents. Lets follow these steps:

1. Go to the folder where your dockeragent images are, in this guide that would be:

        cd c:\dockeragent

2. Lets create a subfolder called kubeconfig.

        mkdir c:\dockeragent\kubeconfig

3. Inside this folder we will copy all the files in this [folder](https://github.com/buys-stash/in-a-box/tree/main/azdevops-selfhostedagents/kubernetes) to the subfolder we just created.
4. After you copied all the required files we are going to login to our AKS Cluster (make sure you are logged in with the Azure CLI and selected the (active) Azure Subscription where you AKS Cluster is deployed in).

        az aks get-credentials --resource-group "rg-selfhostedagents" --name "aks-selfhostedagents"

5. So we are in the subfolder kubeconfig and we are logged into our AKS Cluster. Lets verify this by checking our recently deployed node pool:

        kubectl get nodes

6. If you verified that our nodepool **azdevops** is visible then you can continue, else check if you followed the previous steps correctly!
7. First we need authentication set-up between our AKS Cluster and the Azure Container Registry so AKS can pull the image we created. Apply the following commands where we replace the NAME OF ACR with your ACR name. After that the commands will retrieve the credentials for the Azure Container Registry and use the first password available to add the required access in our AKS Cluster.  

        $ACR_NAME = "NAME OF ACR"
        $REGISTRY_URL = "$ACR_NAME.azureacr.io"
        $ACR_CREDENTIALS = az acr credential show -n $ACR_NAME | ConvertFrom-Json
        $ACR_USERNAME = $ACR_CREDENTIALS.username
        $ACR_PASSWORD = $ACR_CREDENTIALS.passwords[0].value
        kubectl create secret docker-registry acr-secret --docker-server=$REGISTRY_URL --docker-username=$ACR_USERNAME --docker-password=$ACR_PASSWORD

8. Before we apply all the recently copied files we need to adjust some files, lets open the Secret.yml and replace the values mentioned between <> and save the file:

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

9. Now lets open the ReplicationController.yml and replace the values mentioned between <> (in this file it is only the image URL pointing to your ACR) and save the file:

        apiVersion: apps/v1
        kind: Deployment
        metadata:
          name: azdevops
          namespace: azdevops-agents
          labels:
            app: azdevops-agent
        spec:
          replicas: 3 #here is the configuration for the actual agent always running
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

9. Lets apply all the files:

        kubectl apply -f Namespace.yml
        kubectl apply -f Secret.yml
        kubectl apply -f ReplicationController.yml

10. Lets verify our current configuration by checking if the namespace **azdevops-agents** is created:

        kubectl get namespaces

11. Lets verify if the secret azdevops is created, but before we do that lets switch the current context to our newly created namespace:

        kubectl config set-context --current --namespace=azdevops-agents
        kubectl get secret

12. Finally lets verify if the deployment azdevops is created:

        kubectl get deployment

13. After the configuration your AKS Cluster should be up and running, deploying Azure DevOps Self-Hosted Agents. To check this:

        kubectl get pods

14. If you see a pod running with the prefix name azdevops-... then go to the Azure DevOps Agent Pool in the Azure DevOps portal and verify an agent is online.

If this is not working please contact me or go to the troubleshooting Part 7.2 of this guide.

## Part 6. Configure KEDA auto-scaling for the Azure DevOps Self-Hosted Agents inside AKS Cluster

So you have an AKS Cluster and Azure Container Registry up and running but also did some configuration to link these two with the proper authentication. Now you need auto-scaling so lets configure this with the following steps:

1. Go to the folder where your dockeragent images are and navigate to the subfolder kubeconfig:

        cd c:\dockeragent\kubeconfig

2. Follow Step 4 in Part 1 to get the Azure DevOps Pool ID of the Pool you want to deploy Azure DevOps Self-Hosted Agents in.
3. Lets prepare our AKS Cluster to configure KEDA (which is the mechanisme to enable auto-scaling of pods). Make sure you are logged into Azure, selected the right Azure Subscription and are logged into your AKS Cluster. Then run these commands:

        az feature register --name AKS-KedaPreview --namespace Microsoft.ContainerService
        az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKS-KedaPreview')].{Name:name,State:properties.state}"
        az provider register --namespace Microsoft.ContainerService
        az aks update `
        --resource-group "rg-selfhostedagents" `
        --name "aks-selfhostedagents" `
        --enable-keda 

4. Lets check if KEDA is enabled on our AKS Cluster:

        az aks show -g "rg-selfhostedagents" --name "aks-selfhostedagents" --query "workloadAutoScalerProfile.keda.enabled" 

5. If KEDA is enabled then we can apply the scaledObject which contains the scaling configuration for our AKS Cluster. Replace the poolID with the ID you retrieved in step 2:

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

6. Apply the configuration:

        kubectl apply -f ScaledObject.yml

7. Retrieve the ScaledObject and check if Ready is set to True and Active is set to False:

        kubectl get ScaledObject

8. To test this you can configure your pipelines to use the Azure DevOps Agent Pool we configured for auto-scaling, look at the snippet below containing a piece of a YAML pipeline:

        jobs:
        - job: "Example of Jobname"
          timeoutInMinutes: 0
          pool: 'YOUR AGENT POOL'

9. After adding the correct pool to the pipeline, run the pipeline multiple times to see if auto-scaling is working (it requires multiple jobs and some patience to see it working correctly on a Windows image). Another option you can try is this loop to start multiple jobs and trigger auto-scaling (replace the values with your own values):

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

# Part 7. What is next?

Yet to be written...

## Part 7.1. Management

## Part 7.2. Troubleshooting

## Part 7.3. Hardening