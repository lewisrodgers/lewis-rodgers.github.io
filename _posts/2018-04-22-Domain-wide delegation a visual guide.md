---
layout: post
title:  "Domain-wide delegation — a visual guide"
category: Cloud Platform
tags: [gsuite, gcp]
hash: Ha52 
---


[overview]: {{ "/public/img/Ha52-overview.png" | absolute_url }}
[actors]: {{ "/public/img/Ha52-actors.png" | absolute_url }}
[responsibility]: {{ "/public/img/Ha52-responsibility.png" | absolute_url }}
[developer]: {{ "/public/img/Ha52-developer.png" | absolute_url }}
[gcp_admin]: {{ "/public/img/Ha52-gcp_admin.png" | absolute_url }}
[gsuite-admin]: {{ "/public/img/Ha52-gsuite_admin.png" | absolute_url }}

An application may want to access user data without the need for manual authorization by that user. This involves authorizing a service account to access data on behalf of the user — also referred to as domain-wide delegation (DwD) of authority.

For a detailed step-by-step guide in enabling DwD see: [Perform G Suite Domain-Wide Delegation of Authority](https://developers.google.com/admin-sdk/directory/v1/guides/delegation)

The steps boil down to this:

1. A service account is created with DwD enabled
2. A json keyfile – or credentials – is downloaded
3. These credentials are used within an application
4. The API (Admin SDK, Calendar, Drive, etc) used plus its scopes are determined at the application level
5. The client ID (generated when the service account was created) and scopes are applied to the G Suite domain via the Admin console

Visually, how the pieces fit together looks something like this...

![overview]

Next, let's see how the setup process might play out in the real world.

## Separation of concerns

At the enterprise level, there'll be multiple teams or individuals responsible for different aspects of GCP and G Suite administration. Which means, there's going to be some level of coordination when it comes setting up DwD for an application that needs it.

There are distinct actors responsible for providing and consuming each of these things. Let's say: 

1. A developer from the App Dev team
2. The GCP admin* from the Operations team
3. And the G Suite domain admininstrator from some other part of IT

_* GCP Admin is a generalized term I'm using to refer to someone who has the permissions to create service accounts, whether it be a GCP Project Owner or a Service Account Admin._

![actors]

And here's an example what the division of responsibilies looks like...

![responsibility]

The Security team would be a factor in this dance as well, but let's pretend everything has the security stamp of approval.

## The Developer

In our scenario, App Dev wants to build an application that accesses user data on the domain. They determine the APIs and list of API scopes to be used. What's missing is the credentials needed for authentication between the application and G Suite domain. 

![developer]

## The GCP Admin

In order to get the credentials, App Dev asks the GCP Admin for a service account with DwD enabled. The GCP Admin enables the proper APIs, chooses a service account, and makes the json keyfile available to the developers.

![gcp_admin]

## The G Suite Admin

Finally, the application is registered with the G Suite domain so that the domain will know how to identify the application.

To do this, the G Suite Admin needs the client ID associated with the service account and API scopes. They'll add these values to the [Manage client API access](https://support.google.com/a/answer/162106?hl=en) page of the G Suite console.

![gsuite-admin]

## Where to go from here

Often, you'll want to verify the DwD configuration is set up correctly. Test with this bare minimum [python script](https://github.com/lewisrodgers/codelabs/tree/master/google-admin-sdk-api) that can be adjusted for your own needs.

## Resources
* [Using OAuth 2.0 for Server to Server Applications](https://developers.google.com/identity/protocols/OAuth2ServiceAccount)
* [Perform G Suite Domain-Wide Delegation of Authority](https://developers.google.com/admin-sdk/directory/v1/guides/delegation)
* [Perform Google Apps domain-wide delegation of authority](https://developers.google.com/+/domains/authentication/delegation)
* [OAuth: Managing API client access](https://support.google.com/a/answer/162106?hl=en)