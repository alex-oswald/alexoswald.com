---
title:  "Creating my static blog site!"
date:   2020-05-06
classes: wide
words_per_minute: 50
---

I've always wanted to create my own blog to share my solutions to different coding problems I've solved over the years. I've finally gotten around to creating one.
After reading a few other developer blog posts I was able to get a solution working for me that I was happy with.

I started with a few requirements:

- Static site generator with Markdown support
- Custom domain
- Fast & secure
- Cheap, less than $50 per year

Another few requirements regarding personal preferences:

- Azure DevOps for version control
- Azure Pipelines
- Blog deployed in Azure



---

### Prerequisites

1. Azure subscription

2. Custom domain

3. 

### Setup storage account in Azure

1. Create a storage account in Azure

2. Enable the Static website as shown below.

![Storage-static-website](/assets/storage-static-website.png)

Write down the `Primary endpoint`. You will need it later.

---

### Create and publish static site

1. [Install Jekyll on Windows](https://jekyllrb.com/docs/installation/windows/)

    - Download and Install Ruby+Devkit

2. [Create new Jekyll site in VSCode](https://jekyllrb.com/docs/)

3. Create project in Azure DevOps

4. Push site to DevOps from VSCode

5. Add the `azure-piplines.yaml` file to the repo. Use this [Azure pipelines yaml](#azure-pipelines-yaml) code to build the site automatically on each push.

6. Create a Release Pipeline.

    - Use the build pipelines artifact.

    - [Sync with Azure storage yaml](#sync-yaml)

    - Purge CDN

      [Purge CDN endpoint](https://docs.microsoft.com/en-us/cli/azure/cdn/endpoint?view=azure-cli-latest#az-cdn-endpoint-purge)

      ```powershell
      az cdn endpoint purge --profile-name $(cdnProfile) --content-paths '/*' --name $(endpointName) --resource-group $(resourceGroup)
      ```

    - Create a Release to publish your site.

7. View your published static site using the `Primary endpoint`.

---

### Configure Azure CDN to enforce HTTPS & setup custom domains

Now we need to configure Azure CDN to enforce HTTPS, and setup custom domains.

- Either cdnverify.alexoswald or just cdnverify worked
- Site is published to www.alexoswald.com
- Redirect alexoswald.com to www.alexoswald.com
- HTTP redirected to HTTPS

---

### Setup special HTTP rules for the CDN endpoint

It is no longer possible to use Azure CDN's free certificates with an apex/root domain.

![Screenshot](/assets/cdn-fail-adding-https-to-root.png)

Change text to lowercase.

![Screenshot](/assets/change-to-lowercase.PNG)

DNS record.

![Screenshot](/assets/dns-record-cdnverify.PNG)

HSTS Header

![Screenshot](/assets/hsts-header.PNG)

HTTP to HTTPS Redirect

![Screenshot](/assets/http-to-https-redirect.PNG)

Redirect Root to WWW

**DID NOT WORK FOR ME**

I had to use GoDaddy to forward the domain to the www subdomain, which was mapped to the Azure CDN endpoint.

![Screenshot](/assets/redirect-root-to-www.PNG)

---

### Releasing a build to the static site container



---

### Site urls

http://alexoswald.com - Redirected to Origin with GoDaddy domain forwarding

https://alexoswald.com - 

http://www.alexoswald.com - Redirected to Origin with CDN HTTP rule

https://www.alexoswald.com - Origin

---

### DevOps Code

#### Azure pipelines yaml

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

#### Sync yaml

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

---

### References

Here are some other developer blog posts that helped me complete this project.

- [https://arlanblogs.alvarnet.com/adding-a-root-domain-to-azure-cdn-endpoint/](https://arlanblogs.alvarnet.com/adding-a-root-domain-to-azure-cdn-endpoint/)
- [https://www.glennprince.com/article/moving-my-site-onto-a-cdn/](https://www.glennprince.com/article/moving-my-site-onto-a-cdn/)
- [https://www.duncanmackenzie.net/blog/azure-cdn-rules/#redirecting-the-root-domain-to-the-www-version](https://www.duncanmackenzie.net/blog/azure-cdn-rules/#redirecting-the-root-domain-to-the-www-version)