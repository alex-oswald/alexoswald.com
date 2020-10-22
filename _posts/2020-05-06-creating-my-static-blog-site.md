---
title:  "Creating a Fast & Secure Blog with Jekyll, Azure Storage & Azure CDN!"
date:   2020-05-06
toc: true
toc_label: 'Outline'
toc_icon: list-alt
toc_sticky: true
---


I've always wanted to create my own blog to share my solutions to different coding problems I've solved over the years. I've finally made time to create one.
After reading a few other developers blog posts I was able to get a solution working for me that I was happy with.

I started with a few requirements:

- Static site generator - I wanted to use a static site generator for its simplicity and speed benefits.
- Custom domain - I've purchased my own domain so I want to make sure I can use it for my blog.
- Fast & secure - I want it to be super fast & be secure. Load times under 500ms would be great. I want the pages to load in the time it takes to transition with some animation.
- Cheap, less than $50 per year - The domain is $20 a year from GoDaddy with privacy. That leaves $30 for hosting and security.

I am a HUGE .NET fan and everything Microsoft related so I will most always utilize their tools.
Another few requirements regarding personal preferences:

- Azure DevOps for version control
- Azure Pipelines for builds and release deployments (no dev deployment)
- Deployed in Azure


---


### Prerequisites

- Azure subscription

- Azure DevOps

- Own a custom domain

- Visual Studio Code (or your preferred, guide uses VSC)

- [Install Jekyll on Windows](https://jekyllrb.com/docs/installation/windows/)

  I'm not going to explain how to install Jekyll. I following their instructions and they worked fine for me.


---


### 1. Setup storage account in Azure

**Create** a storage account in Azure

![create-storage-account](/assets/images/2020-05-06/create-storage-account.png)

In the storage account, go to **Static website**. Set the `Static website`
option to `Enabled`.

![storage-static-website-enable](/assets/images/2020-05-06/storage-static-website-enable.png)

**Save** the change.

![storage-static-website](/assets/images/2020-05-06/storage-static-website.png)

Write down the `Primary endpoint`. You will need it later.


---


### 2. Create project in Azure DevOps

Login to your DevOps account [https://dev.azure.com/](https://dev.azure.com/).

Add a new project, making sure *Version control* is set to **Git**.

Navigate to **Repos** > **Files**.

Now we are going to **Clone in VS Code** for next steps


---


### 3. Generate the site with Jekyll

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


### 4. Create a build pipeline

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


### 5. Create a Release Pipeline

Our site is built and stored in DevOps as an Artifact, waiting to be deployed.

Navigate to **Pipelines** > **Releases** and click **New pipeline**.

![create-release-pipeline1](/assets/images/2020-05-06/create-release-pipeline1.png)

Select **Empty job**

![create-release-pipeline2](/assets/images/2020-05-06/create-release-pipeline2.png)

Click **X** to close the Stage pane.

![create-release-pipeline3](/assets/images/2020-05-06/create-release-pipeline3.png)

Click **Add an artifact**.

Choose *Build*, select your *Project*, then select the build pipeline we created previously as the *Source*. Click **Add**.

![create-release-pipeline4](/assets/images/2020-05-06/create-release-pipeline4.png)

We have now added the artifact so it can be used by the pipeline.

Under *Stage 1*, click **1 job, 0 task**.

![create-release-pipeline5](/assets/images/2020-05-06/create-release-pipeline5.png)

#### Add tasks

We have two new tasks to add.

1. The first is to Sync the files in the Artifact with your static website container in the Azure Storage account specified. Click the **+** to add a new task.

![create-release-pipeline6](/assets/images/2020-05-06/create-release-pipeline6.png)

Add an `Azure CLI` task.

![create-release-pipeline7](/assets/images/2020-05-06/create-release-pipeline7.png)

I updated the *Display name* to **Sync files**, set my Azure subscription and changed the *Script Location* to *Inline script*. Set the inline script to the below.

```powershell
az storage blob sync --source $(source) --container $(containerName) --account-name $(storageAccount) --auth-mode key --account-key $(key)
```

![create-release-pipeline8](/assets/images/2020-05-06/create-release-pipeline8.png)

2. The second is to Purge the CDN of all cached files so the new site is pulled and cached.

Add another `Azure CLI` task.

I updated the *Display name* to **Purge CDN**, set my Azure subscription and changed the *Script Location* to *Inline script*. Set the inline script to the below.

```powershell
az cdn endpoint purge --profile-name $(cdnProfile) --content-paths /* --name $(endpointName) --resource-group $(resourceGroup)
```

![create-release-pipeline9](/assets/images/2020-05-06/create-release-pipeline9.png)

#### Add variables

Both of our tasks use multiple variables to define. While still editing Release, click **Variables**.

Add the following variables:

**Sync files**

*source*: Folder name in the artifact that contains the site

*containerName*:  $web (set by Azure)

*storageAccount*:  The name of your storage account

*key*: Access key to your storage account

**Purge CDN**

*cdnProfile*: The name of the CDN Profile used by the Endpoint resource

*endpointName*: The name of the Endpoint

*resourceGroup*: The name of the Resource Group that contains the Endpoint

![create-release-pipeline10](/assets/images/2020-05-06/create-release-pipeline10.png)

**Make sure to Save your pipeline**

#### Create a release

In the top right next to the Save button you just clicked, click **Create release**, after the dialog shows, click **Create**.

This process may take a few minutes. Once complete, open a browser and navigate to the storage accounts `Primary endpoint` you wrote down previously. Your site should now be deployed!

As it stands, you can release a new version of your site by:

1. Making changes in VS Code
2. Committing changes
3. Wait for build pipeline to succeed
4. Create release pipeline
5. View new deployment in a browser

The web address to access the site is currently the `Primary endpoint` for the Storage accounts web container.


---


### 6. Configure Azure CDN to enforce HTTPS & setup custom domains

Now we want to utilize Azure CDN to cache the static site so it is fast & secure for viewers.

>**IMPORTANT**  
>I was able to get my apex domain added as a custom domain to my Azure CDN endpoint, but Azure no longer supports using their free SSL certificates for apex domains. I can't purchase one because it would blow my yearly budget. The option I came up with involves setting up Azure CDN using the `www` subdomain and then forwarding requests to the apex domain to the www subdomain. While I personally dislike having to use the `www` subdomain at all, it is providing me a free SSL certificate.

![azure-cdn-apex-https-fail](/assets/images/2020-05-06/azure-cdn-apex-https-fail.png)

#### Create Azure CDN endpoint

While viewing the storage account resource, under **Blob service**, go to **Azure CDN**.

From here, we are going to create a new `Endpoint`.

Create a new **CDN profile** with the name of your choosing.

Make sure the **Pricing tier** is set to *Premium Verizon* so we can set HTTP rules later.

Enter the **CDN endpoint name**.

Click **Create**.

![create-azure-cdn-endpoint](/assets/images/2020-05-06/create-azure-cdn-endpoint.png)

Navigate to the newely created resource.

Navigate to **Settings** > **Origin** and deselect **HTTP**. We only want to allow HTTPS.

![endpoint-disable-http](/assets/images/2020-05-06/endpoint-disable-http.png)

#### Add DNS records to domain

In order to setup our custom domains Azure needs to verify you own the domain so we need to setup some CNAME records that Azure can verify.

Add a CNAME with **Host** `www` and **Value** `myblog-endpoint.azureedge.net`.

I'm using GoDaddy for my domains.

![cname-records](/assets/images/2020-05-06/cname-records.png)

#### Add custom domain to the CDN endpoint

Navigate to the Azure CDN Endpoint resource.

Navigate to **Settings** > **Custom domains** > **+ Custom domain**

![add-custom-domain-to-endpoint](/assets/images/2020-05-06/add-custom-domain-to-endpoint.png)

Fill in your `www` subdomain. If Azure is able to verify your CNAME record you will see the green check, otherwise there will be a red X.

![add-a-custom-domain](/assets/images/2020-05-06/add-a-custom-domain.png)

Now we need to enable HTTPS for the hostname.

Click your custom domain.

![endpoint-custom-domains](/assets/images/2020-05-06/endpoint-custom-domains.png)

Set **Custom domain HTTPS** to **On**.

I am using Azure's free certificates so I set **Certificate management type** to **CDN managed**.

Then click **Save**.

![enable-custom-domain-https](/assets/images/2020-05-06/enable-custom-domain-https.png)

In my experience, the certificate deployment process can take a few hours to complete.

Once complete, the screen should look like the below.



![https-successfully-enabled](/assets/images/2020-05-06/https-successfully-enabled.png)


#### Complete custom domain setup


![godaddy-domain-forwarding](/assets/images/2020-05-06/godaddy-domain-forwarding.png)


---


### 7. Setup special HTTP rules for the CDN endpoint

Change text to lowercase.

![Screenshot](/assets/images/2020-05-06/change-to-lowercase.png)


HSTS Header

![Screenshot](/assets/images/2020-05-06/hsts-header.png)

HTTP to HTTPS Redirect

![Screenshot](/assets/images/2020-05-06/http-to-https-redirect.png)

Redirect Root to WWW

**DID NOT WORK FOR ME**

This wasn't working for me. I had to use GoDaddy to forward the domain to the www subdomain, which was mapped to the Azure CDN endpoint.

![Screenshot](/assets/images/2020-05-06/redirect-root-to-www.png)


---


### 8. Releasing a build to the static site container



---

### Appendixes


#### Appendix A: Site urls

http://alexoswald.com - Redirected to Origin with GoDaddy domain forwarding

https://alexoswald.com - 

http://www.alexoswald.com - Redirected to Origin with CDN HTTP rule

https://www.alexoswald.com - Origin


#### Appendix B: DevOps Code

##### Azure pipelines yaml

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

##### Sync files

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

##### Purge CDN

```yaml
# Purges the CDN's cache so it has to fetch new (updated)
# content from the storage container
steps:
- task: AzureCLI@1
  displayName: 'Purge CDN'
  inputs:
    azureSubscription: 'Visual Studio Professional (3a69699e-acaf-48c5-a4a3-d506bb04d4a6)'
    scriptLocation: inlineScript
    inlineScript: 'az cdn endpoint purge --profile-name $(cdnProfile) --content-paths /* --name $(endpointName) --resource-group $(resourceGroup)'
```

---


#### Appendix C: References

Here are some other developer blog posts that helped me complete this project.

- [https://arlanblogs.alvarnet.com/adding-a-root-domain-to-azure-cdn-endpoint/](https://arlanblogs.alvarnet.com/adding-a-root-domain-to-azure-cdn-endpoint/)
- [https://www.glennprince.com/article/moving-my-site-onto-a-cdn/](https://www.glennprince.com/article/moving-my-site-onto-a-cdn/)
- [https://www.duncanmackenzie.net/blog/azure-cdn-rules/#redirecting-the-root-domain-to-the-www-version](https://www.duncanmackenzie.net/blog/azure-cdn-rules/#redirecting-the-root-domain-to-the-www-version)