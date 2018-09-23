---
layout: post
current: post
cover: assets/images/Ha52-overview.png
navigation: True
title: Domain-wide delegation — a visual guide
tags: [g suite]
class: post-template
subclass: 'post'
hash: Ha52 
---

An application that we build may need to access information that belongs to an employee, or do something on their behalf. Things like, sending an email for them, or delete some of their calendar events. Usually, the user has a chance to review what the application will have access to — or **scope** — and choose whether or not to allow it. You'll know it as this screen:

![authorization]

Sometimes a business case calls for a tool that should access employee data without the need for manual authorization by that employee. We use domain-wide delegation (DwD) of authority to achieve this.

For a detailed step-by-step guide in enabling DwD see: [Perform G Suite Domain-Wide Delegation of Authority](https://developers.google.com/admin-sdk/directory/v1/guides/delegation), but it essentially boils down to this:

1. A service account is created with DwD enabled
2. A json keyfile – or credentials – is downloaded
3. These credentials are used within an application
4. The API (Admin SDK, Calendar, Drive, etc) used plus its scopes are determined at the application level
5. The client ID (generated when the service account was created in step 1) and scopes are applied to the G Suite domain via the Admin console

Visually, how the pieces fit together looks something like this...

![overview]

Ok, now let's see how the process might play out when separate teams are responsible for different steps.

## Separation of concerns

At the enterprise level, there'll be multiple teams or individuals responsible for different aspects of GCP and G Suite administration. Which means, there's going to be some level of coordination when it comes to setting up DwD for an application that needs it.

There are distinct actors responsible for providing and consuming each of these things. Let's say: 

1. A developer from the App Dev team
2. The GCP admin* from the Operations team
3. And the G Suite domain administrator from some other part of IT

> *GCP admin is a generalized term I'm using to refer to someone who has the permissions to create service accounts, whether it be a GCP Project Owner or a Service Account Admin.

![actors]

And here's an example what the division of responsibilities looks like...

![responsibility]

The Security team would be a factor in this dance as well, but let's pretend everything has the security stamp of approval.

## The Developer

In our scenario, App Dev wants to build an application that accesses user data on the domain. They determine the APIs and list of API scopes to be used. What's missing is the credentials needed for authentication between the application and G Suite domain. 

![developer]

## The GCP admin

In order to get the credentials, App Dev asks the GCP admin for a service account with DwD enabled. The GCP admin enables the proper APIs, chooses (or creates) a service account, and makes the credentials available to the developers.

> Sidenote: Security's Trust No One policy means it doesn't like the idea of handing over credentials as plain text to anyone outside the Operations team. So, the GCP admin might use Cloud KMS to encrypt the credentials. I've left this out of the diagrams so it doesn't over complicate them.

![gcp_admin]

## The G Suite admin

Finally, the application is registered with the G Suite domain so that the domain will know how to identify the application.

To do this, the G Suite admin needs the client ID associated with the service account and API scopes. They'll add these values to the [Manage client API access](https://support.google.com/a/answer/162106?hl=en) page of the G Suite console.

![gsuite-admin]

## Where to go from here

Often, you'll want to verify the DwD configuration is set up correctly. Test with this bare minimum [python script](https://github.com/lewisrodgers/codelabs/tree/master/google-admin-sdk-api) that can be adjusted for your own needs.

## Resources
* [Using OAuth 2.0 for Server to Server Applications](https://developers.google.com/identity/protocols/OAuth2ServiceAccount)
* [Perform G Suite Domain-Wide Delegation of Authority](https://developers.google.com/admin-sdk/directory/v1/guides/delegation)
* [Perform Google Apps domain-wide delegation of authority](https://developers.google.com/+/domains/authentication/delegation)
* [OAuth: Managing API client access](https://support.google.com/a/answer/162106?hl=en)


[authorization]: {{ "/assets/images/Ha52-authorization.png" | absolute_url }}
[overview]: {{ "/assets/images/Ha52-overview.png" | absolute_url }}
[actors]: {{ "/assets/images/Ha52-actors.png" | absolute_url }}
[responsibility]: {{ "/assets/images/Ha52-responsibility.png" | absolute_url }}
[developer]: {{ "/assets/images/Ha52-developer.png" | absolute_url }}
[gcp_admin]: {{ "/assets/images/Ha52-gcp_admin.png" | absolute_url }}
[gsuite-admin]: {{ "/assets/images/Ha52-gsuite_admin.png" | absolute_url }}