---
title: "Outlook Rule for 'on behalf of' Emails"
slug: outlook-rule-for-on-behalf-of-emails
aliases:
- /2021/11/17/outlook-rule-for-on-behalf-of-email.html
date: 2021-11-17
---


I get a lot of different alert emails, from Azure DevOps, AWS, etc. Setting up rules to move these emails to different folders is important for my sanity. Alert emails I typically get come from a central source, and the *on behalf of* email is from the root source, like Azure DevOps or AWS. This post answers the question:

## How do I setup an Outlook rule to filter *on behalf of* emails?

- Open the **Rules and Alerts** window in Outlook
- Click **New Rule...**
- Click **Next** to skip to the advanced setup

![outlook_rule_1](outlook_rules_1.png)

Unfortuantely, we can't create a rule for *on behalf of* email address using the `from people or public group`. The address is stored in the email header though, so we can search the header for the address.

- Select **with specific words in the message header**
- Click **specific words** in Step 2 to add the email address you want to filter on

![outlook_rule_2](outlook_rules_2.png)

- Click **Ok** to exit the **Search Text** dialog

![outlook_rule_3](outlook_rules_3.png)

- Select the folder you want to move emails to and click **Finish**

