---
layout: post
title:  "GitHub Tips and Tricks"
date:   2022-08-09 17:00:00 +0200
categories: [github, azure active directory, branching, permissions]
---

Do you want to know more around Identity in GitHub Actions, filters in GitHub Projects and many more then keep on reading and check the Table of Contents!

* TOC
{:toc}

# GitHub App

So lets talk about the GitHub App.

## When to use it?

Decide for yourself if you want/need to use it:

![image](../.attachments/intro-to-apps-flow.png)

## Azure App Registration with Federated Credentials

So if you want to implement this with an Azure App Registration with Federated Credentials, check out below guide:



## Limitation(s)

### Azure App Registration with Federated Credentials

So when using a GitHub App with an Azure App Registration with Federated Credentials then be aware of the limited lifetime of the access token which is set to 10 minutes. So each time a command is running longer then the lifetime of the access token this is not something you want to implement (yet).

Off course a workaround could be to get the token at some point in your job or step to bypass this limitation.

# Sync between repositories



# Cannot create file

In the case of running a PowerShell script you can get the error when creating a new file:

``
error: unable to create file
``

To fix this:

``
git config core.protectNTFS false
``