---
layout: post
title:  "Reverse Engineer API Calls in Web Applications"
date:   2022-12-06 17:00:00 +0200
categories: [azure devops, rest api, reverse engineering]
---

Do you want to know how API Calls in Web Applications work and how to use them to build your own application? Keep on reading and I will tell you more about **what** and **how** to do this.

* TOC
{:toc}

# What is an API Call?

Lets not try to write an explanation our self but lets get one that I find the correct definition for this:

Application programming interfaces (APIs) are a way for one program to interact with another. API calls are the medium by which they interact. An API call, or API request, is a message sent to a server asking an API to provide a service or information.

# How to reverse engineer an API call for a Web Application?

So first **why** did I write this at all? Because I not all documentation is/was available for the Azure DevOps REST API. But in the case you don't have the documentation you must become creative somehow, right?

For the example below I used my own Azure DevOps Organization and Contoso Project.

## Taking a look at our example

In our example we are going to retrieve in Azure DevOps how to create a Variable Group without consulting the documentation.

Open up 