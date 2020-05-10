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

- Azure subscription

- Azure DevOps

- Custom domain

- Visual Studio Code

- [Install Jekyll on Windows](https://jekyllrb.com/docs/installation/windows/)

  I'm not going to explain how to install Jekyll. I following their instructions and they worked fine for me.


---


### Setup storage account in Azure

**Create** a storage account in Azure

In the storage account, go to **Static website**. Set the `Static website`
option to `Enabled`.

![Storage-static-website](/assets/images/2020-05-06/storage-static-website.png)

Write down the `Primary endpoint`. You will need it later.


---


### Create project in Azure DevOps

Login to your DevOps account [https://dev.azure.com/](https://dev.azure.com/).

Add a new project, making sure *Version control* is set to **Git**.

Navigate to **Repos** > **Files**.

Now we are going to **Clone in VS Code** for next steps


---


### Generate the site with Jekyll

Once your repo folder is open in VS Code, lets open a Terminal window and run some commands.

The first command creates the site in our repo folder.

```powershell
jekyll new ./
```

What was once an empty folder, should now be populated with the following.

![jekyll-new-file-structure](/assets/images/2020-05-06/jekyll-new-file-structure.png)

Now we are going to build and run the site to check it out.

Run the command.

```powershell
bundle exec jekyll serve
```

You should be able to ctrl + click to open your default browser and view your site.

![server-address](/assets/images/2020-05-06/server-address.png)


---


### Create a build pipeline

We are going to create a build pipeline in Azure DevOps so the site is built each time a push is made to the master or a tag is created. The built site is stored as an Artifact and can be used in other pipelines.

Navigate to **Pipelines** > **Pipelines** and click **Create Pipeline**.

![create-build-pipeline1](/assets/images/2020-05-06/create-build-pipeline1.png)

You will be asked where your code is. Select **Azure Repos Git**.

![create-build-pipeline2](/assets/images/2020-05-06/create-build-pipeline2.png)

Then select your repository.

![create-build-pipeline3](/assets/images/2020-05-06/create-build-pipeline3.png)

We are going to wipe out the example yaml file so you can select **Ruby** or **Starter pipeline**.

![create-build-pipeline4](/assets/images/2020-05-06/create-build-pipeline4.png)

I picked the **Starter pipeline**. Now replace it all with this [Azure pipelines yaml](#azure-pipelines-yaml) code.

![create-build-pipeline5](/assets/images/2020-05-06/create-build-pipeline5.png)

Click **Save and run**, specify commit message and options, then click **Save and run** again.

Lets let the job run and wait for the result.


---


### Create a Release Pipeline

Our site is built and stored in DevOps as an Artifact waiting to be deployed.

Navigate to **Pipelines** > **Releases** and click **New pipeline**.

![create-release-pipeline1](/assets/images/2020-05-06/create-release-pipeline1.png)

Select **Empty job**

![create-release-pipeline2](/assets/images/2020-05-06/create-release-pipeline2.png)

Click **X** to close the Stage pane.

![create-release-pipeline3](/assets/images/2020-05-06/create-release-pipeline3.png)

Click **Add an artifact**. Choose *Build*, select your *Project*, then select the build pipeline we created previously as the *Source*. Click **Add**.

![create-release-pipeline4](/assets/images/2020-05-06/create-release-pipeline4.png)

We have now added the artifact so it can be used by the pipeline. Under *Stage 1*, click **1 job, 0 task**.

![create-release-pipeline5](/assets/images/2020-05-06/create-release-pipeline5.png)






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

![Screenshot](/assets/images/2020-05-06/cdn-fail-adding-https-to-root.png)

Change text to lowercase.

![Screenshot](/assets/images/2020-05-06/change-to-lowercase.png)

DNS record.

![Screenshot](/assets/images/2020-05-06/dns-record-cdnverify.PNG)

HSTS Header

![Screenshot](/assets/images/2020-05-06/hsts-header.png)

HTTP to HTTPS Redirect

![Screenshot](/assets/images/2020-05-06/http-to-https-redirect.png)

Redirect Root to WWW

**DID NOT WORK FOR ME**

I had to use GoDaddy to forward the domain to the www subdomain, which was mapped to the Azure CDN endpoint.

![Screenshot](/assets/images/2020-05-06/redirect-root-to-www.png)


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