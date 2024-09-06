---
title: "Day 1 Dev Log"
date: 2024-09-01
draft: false
github_link: "https://github.com/adamwhiles"
author: "Adam"
tags:
  - blog
image: /images/blog/cr-challenge-day1.png
description: "Day 1 Dev Log"
toc:
---

## Day 1 Dev Log
For day 1 of the cloud resume challenge I decided to focus on creating a very simple HTML site and getting it up on Azure. Once the basic site was operational as a static site, I wanted to the Github action going so that I can quickly make changes and publish going forward. I believe that this will help things move a bit faster.

## Initial Setup
1. Created simple HTML page
2. Created blob storage account
	1. enable static website
	2. set index.html as default
3. Push site to $web container of the storage account
4. Setup Git repo and sync HTML site with that.

## CI/CD Pipeline with GitHub
1. Create Service principal
	1. Run ``` az connect ```
	2. Set the correct subscription by running ```az account set --subscription prod ```
	3. Create service principal, modify the subscription id and resource group name
		1.  ```az ad sp create-for-rbac --name "myML" --role contributor --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name> --json-auth```
		2. Copy the JSON output and note if somewhere as you will need this later.
	4. Configure GitHub Secret
		1. Go to the GitHub repo, then Settings -> Security -> Secrets and Variables -> Actions
		2. Choose New Repository Secret
			1. Name it ```AZURE_CREDENTIALS```
			2. For the value(secret), paste in the JSON output from the command earlier.
	5. Add GitHub Action Workflow
		1. Click Actions at the top of your GitHub repo
		2. Select the option to setup a workflow yourself
		3. Paste the code below and make sure you update the storage account name to match yours.

```
name: Blob storage website CI

on:
    push:
        branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
        
    - name: upload to blob
      uses: azure/CLI@v1
      with:
        inlineScript: |
          az storage blob sync --account-name resumechallengetake2 --container '$web' --source './'

    - name: logout
      run: |
        az logout
      if: always()
```