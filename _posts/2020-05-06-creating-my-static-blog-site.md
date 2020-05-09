---
author: Alex Oswald
title:  "Creating my static blog site!"
date:   2020-05-06
---

# My own blog

I've always wanted to create my own blog to share my solutions to different coding problems I've solved over the years. I've finally gotten around to creating one. And I did it with my favorite tools! __Here is my adventure...__

## My requirements

- Use Azure DevOps for version control and CI/CD
- Use Azure storage's static website feature
- Static site generator
- Custom domain

---

## References

- https://arlanblogs.alvarnet.com/adding-a-root-domain-to-azure-cdn-endpoint/
- https://www.glennprince.com/article/moving-my-site-onto-a-cdn/
- https://www.duncanmackenzie.net/blog/azure-cdn-rules/#redirecting-the-root-domain-to-the-www-version

---

# Setup storage account in Azure

Create a storage account in Azure

Enable the Static website. Save the `Primary endpoint`. You will need it later.

![Screenshot](/assets/storage-static-website.PNG)
Enable static website

Copy `Primary endpoint`

---

# Create and publish static site

1. [Install Jekyll on Windows](https://jekyllrb.com/docs/installation/windows/)

    - Download and Install Ruby+Devkit

2. [Create new Jekyll site in VSCode](https://jekyllrb.com/docs/)

3. Create project in Azure DevOps

4. Push site to DevOps from VSCode

5. Add the `azure-piplines.yaml` file to the repo. Use this [Azure pipelines yaml](#Azure-pipelines-yaml) code to build the site automatically on each push.

6. Create a Release Pipeline.

    - Use the build pipelines artifact.

    - [Sync with Azure storage yaml](#Sync-yaml)

    - Purge CDN

    - Create a Release to publish your site.

7. View your published static site using the `Primary endpoint`.

---

# Configure Azure CDN to enforce HTTPS & setup custom domains

Now we need to configure Azure CDN to enforce HTTPS, and setup custom domains.


---

# Setup special HTTP rules for the CDN


---

# Releasing a build to the static site container

[Purge CDN endpoint](https://docs.microsoft.com/en-us/cli/azure/cdn/endpoint?view=azure-cli-latest#az-cdn-endpoint-purge)

```powershell
az cdn endpoint purge --profile-name $(cdnProfile) --content-paths '/*' --name $(endpointName) --resource-group $(resourceGroup)
```







- Either cdnverify.alexoswald or just cdnverify worked
- Site is published to www.alexoswald.com
- Redirect alexoswald.com to www.alexoswald.com
- HTTP redirected to HTTPS



![Screenshot](/assets/cdn-fail-adding-https-to-root.png)

![Screenshot](/assets/change-to-lowercase.PNG)

![Screenshot](/assets/dns-record-cdnverify.PNG)

![Screenshot](/assets/hsts-header.PNG)

![Screenshot](/assets/http-to-https-redirect.PNG)

![Screenshot](/assets/redirect-root-to-www.PNG)



---

## Azure pipelines yaml

`azure-pipelines.yaml`

```yaml
# Trigger pipeline on any push to master and any tag creations
trigger:
  branches:
    include:
    - master
  tags:
    include: ['*']

# Build on Ubuntu pool
pool:
  vmImage: 'ubuntu-latest'

steps:
# Use Ruby
- task: UseRubyVersion@0
  displayName: 'Use Ruby >= 2.5'
  inputs:
    versionSpec: '>= 2.5'

# Install Jekyll
- script: 'gem install jekyll bundler'
  displayName: 'Install Jekyll and bundler'

# Install Jekyll dependency
- script: 'bundle install'
  displayName: 'Install Gems'

# Run Jekyll and build the site
- script: 'bundle exe jekyll build'
  displayName: Build

# Copy packaged site to the staging directory for publishing
- task: CopyFiles@2
  displayName: 'Copy "_site" to staging directory'
  inputs:
    SourceFolder: '_site'
    TargetFolder: '$(Build.ArtifactStagingDirectory)'

# Publish the artifact
- task: PublishBuildArtifacts@1
  displayName: 'Publish Artifact: _site'
  inputs:
    ArtifactName: '_site'
```

## Sync yaml

```yaml
# This code syncs with the storage blob you specify
# It won't delete all files, then upload. It will 
# upload only what is needed, and only delete what
# is needed (less operations).
steps:
- task: AzureCLI@1
  displayName: 'Sync files'
  inputs:
    azureSubscription: '$(subscription)'
    scriptLocation: inlineScript
    inlineScript: 'az storage blob sync --source $(source) --container $(containerName) --account-name $(storageAccount) --auth-mode key --account-key $(key)'
    workingDirectory: '$(System.DefaultWorkingDirectory)/_MyBlog'
```