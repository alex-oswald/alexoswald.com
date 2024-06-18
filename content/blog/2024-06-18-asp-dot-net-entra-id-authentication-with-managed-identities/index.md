---
title: "ASP.NET Entra ID Authentication with Managed Identities"
slug: asp-dot-net-entra-id-authentication-with-managed-identities
date: 2024-06-18
---

Security is critical these days and the removal of secrets is always a great thing. This blog post explains how to use Entra ID authentication with an ASP.NET app **without** client secrets using managed identities in Azure.

At a high level, the following needs to be completed for this to work.

1. Create a Using Assigned Managed identity in Azure
2. Managed identity added as an identity to the web app
3. Managed identity added to the Entra ID app using federated identity credentials
4. Config section added to the web apps environment variables

> Please note, this ONLY works for resources published in Azure. You cannot use this locally.

# Create the Managed Identity

Lets first create a **User Assigned Managed Identity** in Azure. I'm going to call it `Entra-Id-Identity`.

![create-managed-identity](create-managed-identity.png)

Once your resource is created, nagivate to it and make a note of its **Client ID**, as we will need that later.

# Create App Registration

Next, we need to create an Entra ID app. Give it a name and click **Register**.

![create-app-registration](create-app-registration.png)

Once our app is created, navigate to **Certificates & secrets**, then **Federated credentials**. Click **Add credential**. We are going to create a FIC with our managed identity. Select your managed identity and give the FIC a name of your choice and click **Add**.

![add-fic](add-fic.png)

You should now have an FIC added to your app.

![fic](fic.png)

# Create your ASP.NET application

Now we will create our application. You should be able to use any type of app that supports managed identities. I have tested this with Azure Container Apps and Azure Web Apps.

I'm going to create a simple Web App.

![create-web-app](create-web-app.png)

This post isn't going to show you how to publish your app to a web app or container app in Azure, but I will show you how to assign the managed identity to the app.

After you've created your app, open the resource and navigate to the **Identity** tab in the Azure portal. Click the **User assigned** tab and add your identity.

![web-app-identity-tab](web-app-identity-tab.png)

# Create an ASP.NET project with the Microsoft Identity Platform

When you create any ASP.NET project with authentication using the Microsoft Identity Platform via a Visual Studio template, you get the default `AzureAd` config section. This section needs to be updated to use the federated identity credential.

![create-asp-net-app-with-identity-platform](create-asp-net-app-with-identity-platform.png)

![program-appsettings](program-appsettings.png)

Of course you need to fill in your `TenantId` and the `ClientId` of your Entra ID app.

To use the FIC from the managed identity you need to add a new `ClientCredentials` section to `AzureAd` like the below. `ManagedIdentityClientId` should be set to the client ID of the managed identity that you created.

```json
"ClientCredentials": [
    {
    "SourceType": "SignedAssertionFromManagedIdentity",
    "ManagedIdentityClientId": "911cdd2a-d63e-472f-a227-78ee1ae1d012",
    "TokenExchangeUrl": "api://AzureADTokenExchange"
    }
]
```

With the new section added.

![client-credentials-section](client-credentials-section.png)

And that is it.
