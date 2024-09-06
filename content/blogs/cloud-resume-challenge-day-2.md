---
title: "Day 2 Dev Log"
date: 2024-09-01
draft: false
github_link: "https://github.com/adamwhiles"
author: "Adam"
tags:
  - blog
image: /images/blog/cr-challenge-day2.png
description: "Day 2 Dev Log"
toc:
---

## Day 2 Dev Log
For day 2 of the cloud resume challenge the focus was on switching over to a Hugo site with a nice theme and connection a custom domain to the site. The custom domain is set up through Azure Front Door.

## Hugo
- Switch from plain HTML site to Hugo
- Setup and customize hugo-profile theme

## GitHub Actions
- Create GitHub Workspace and Secret for the Azure Blob Connection String
- Modify GitHub action to work with Hugo (code at bottom of post)

## Domain
Connected adamwhiles.com domain name
- First setup Front Door CDN
	- From the storage account, go to security+networking and click front door and cdn.
	- Fill out the required details and keep the default options, click create
- Once the Front Door resource is created, go to it
	- Under settings, choose domains and add a domain.
	- Ownership will need to be validated and a CNAME added
	- Add CNAME to point to Front Door URL
	- Add www.adamwhiles.com to Azure Front Door and complete ownership validation

```
# This is a basic workflow to help you get started with Actions

name: Build and deploy Hugo static webpage to Azure blob

# Controls when the workflow will run
on:
  # Triggers the workflow on push but only for the main branch
  push:
    branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    environment: build-environment
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:

      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0
    
        # Uses code from user peaceiris to setup a hugo docker for us to build our website with
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v3
        with:
          hugo-version: 'latest'
          extended: true

      # Build our website using hugo
      - name: Build site
        run: hugo --minify

      # Connect to azure and upload the /public directory to our $web storage blob using code from user bacongobbler
      - name: Deploy to Azure Blob Storage
        uses: bacongobbler/azure-blob-storage-upload@main
        with:
            source_dir: public #Hugo build generates this file after build
            container_name: $web
            connection_string: ${{ secrets.BLOB_STORAGE_CONNECTION_STRING }}
            sync: 'true'
  ```