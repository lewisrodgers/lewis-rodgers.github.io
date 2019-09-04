---
layout: post
title: "Identifying user-managed and Google-managed service accounts"
date: 2019-09-01 00:00:00 -0400
tags: [GCP]
---


User-managed service accounts are identifiable by these email formats:

| Service | Service account |
|-|-|
| Compute Engine | PROJECT_NUMBER-compute@developer.gserviceaccount.com |
| App Engine | PROJECT_NUMBER@appspot.gserviceaccount.com |
| User defined | SERVICE_ACCOUNT_NAME@PROJECT_ID.iam.gserviceaccount.com |

Service accounts displayed in the **Service Accounts** section of the Cloud Console are considered to be _user-managed_. And, there’s a `gcloud` command for listing these user-managed service accounts.

```bash
gcloud iam service-accounts list
```

The only GCP services mentioned in the documentation—that generate user-managed service accounts automatically—are Compute Engine and App Engine[^1]. So I assume these are the only services that do this. All other service accounts, that aren’t explicitly created by _you_, are Google-managed.

A Google-managed service account is displayed in the **IAM** section of the Cloud Console and are not displayed in the Service Accounts section. These are generated automatically when GCP services are enabled. For example, when you deploy a Cloud Function for the first time or enable an API from the **API Library** section.

You don’t use the gcloud command `gcloud iam service-accounts list` to list Google-managed service accounts. To list Google-managed service accounts in your project, get the IAM policy instead.

```bash
gcloud projects get-iam-policy $PROJECT_ID
```

And with some flattening and filtering you can list just the service accounts.

```bash
gcloud projects get-iam-policy $PROJECT_ID \
--flatten="bindings[].members[]" \
--format="table(bindings.members)" \
--filter="bindings.members:serviceAccount"
```

## Identifying Google-managed service accounts

There are a few different ways to tell a service account is Google-managed: through email patterns, help text, and by checking the project's IAM policy (the most straight forward and definitive method). 

### Email patterns

Most Google-managed service accounts follow this email pattern:

```
service-PROJECT_NUMBER@SERVICE_NAME.iam.gserviceaccount.com
```

Here are some examples with this pattern:

| Service | Service account |
|-|-|
| Compute Engine | service-PROJECT_NUMBER@compute-system.iam.gserviceaccount.com |
| Cloud Filestore | service-PROJECT_NUMBER@cloud-filer.iam.gserviceaccount.com |
| Cloud AI Platform | service-PROJECT_NUMBER@cloud-ml.google.com.iam.gserviceaccount.com |

But, this isn't always the case:

| Service | Service account |
|-|-|
| Google APIs | PROJECT_NUMBER@cloudservices.gserviceaccount.com |
| Cloud Build |  PROJECT_NUMBER@cloudbuild.gserviceaccount.com |
| Firebase | firebase-service-account@firebase-sa-management.iam.gserviceaccount.com |

### Help text

Also, some service accounts have help text that tell us that a service account is Google-managed.

{% include image.html name="help-text.png" alt="Cloud Functions service account in IAM members list" caption="Hovering over the question mark (?) explicitly tells us that the service account is Google-managed." %}

And, like the email patterns, this isn't always the case.

{% include image.html name="no-help-text.png" alt="Cloud Storage service account in IAM members list" caption="No help text here." %}

### IAM policy

The email patterns and help text aren't consistent, but the rule of thumb is: if the service account is listed in the project's IAM policy and doesn't show up in the Service Accounts section, then it's Google-managed.

## Resources

- [List of GCP generated service accounts](https://github.com/lewisrodgers/google-managed-service-accounts)


[^1]: [https://cloud.google.com/iam/docs/service-accounts#user-managed_service_accounts](https://cloud.google.com/iam/docs/service-accounts#user-managed_service_accounts){:target="_blank"}
