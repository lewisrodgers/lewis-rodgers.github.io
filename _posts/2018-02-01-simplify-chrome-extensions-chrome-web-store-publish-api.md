---
layout: post
title:  "Simplify Chrome extension publishing with the Chrome Web Store Publish API"
category: Development
tags: [chrome extension, bitbucket, continuous delivery]
hash: tD19
---

I was on a consulting engagement with a real estate development company in California. One of the projects involved building out a few Chrome extensions. 

For this effort, the team included two developers and myself. I took on a DevOps role so that the others could focus on code. This meant I was responsible for versioning, packaging, uploading, and publishing the extensions to the Chrome Web Store (CWS, or just store). It allowed me to control what would be released for testing, QA, and to stakeholders.

The extensions were relatively small applications, and figured I could handle the DevOps manually. But that was a mistake. Turns out, juggling multiple extensions is really tricky. 

In fact, at one point I packaged up the wrong extension files by mistake because I was in the wrong git branch. 

But, with automated deployments you can avoid these mistakes. In this post, I’ll show you how to leverage the CWS Publish API and combine it with a continuous deployment tool like Travis CI or — in this case — Bitbucket Pipelines.

## Prerequisites

Make sure the following command line tools are installed on your machine:

* curl - command-line tool for transferring data specified with URL syntax.
* zip - command-line tool for archiving files.
* jq - lightweight and flexible command-line JSON processor.


## Chrome Web Store API

What does the CWS Publish API do?

> The Chrome Web Store Publish API provides a set of REST endpoints for programmatically creating, updating, and publishing items in the Chrome Web Store.

It gives us a way to update an existing store item, and then publish that store item using code instead of visiting the Developer Dashboard. Here’s how the endpoints look:

Upload package for existing store item:

```sh
$ curl \
-H "Authorization: Bearer $TOKEN"  \
-H "x-goog-api-version: 2" \
-X PUT \
-T $FILE_NAME \
-v \
https://www.googleapis.com/upload/chromewebstore/v1.1/items/$APP_ID
```

Publish a store item:

```sh
$ curl \
-H "Authorization: Bearer $TOKEN"  \
-H "x-goog-api-version: 2" \
-H "Content-Length: 0" \
-X POST \
-v \
https://www.googleapis.com/chromewebstore/v1.1/items/$APP_ID/publish
```

These endpoints can be run using curl in the terminal, but we need a few things: an access token, our packaged extension, and the app ID of the extension we want to update or publish. Let’s address these in the next sections.

## Access Token

First, take care of authorization. There’s a well documented guide in the official chrome developer docs on how to do this. So, head over to [Using the Chrome Web Store Publish API](https://developer.chrome.com/webstore/using_webstore_api#beforeyoubegin) and do the steps under the section titled [Before you begin](https://developer.chrome.com/webstore/using_webstore_api#beforeyoubegin).

Come back after you have your access and refresh tokens.

## App ID

The ID of your extension can be found either in the store or the [Developer Dashboard](https://chrome.google.com/webstore/developer/dashboard). Go to the dashboard. Find the extension you want to update, or publish. Then, open the **More info** panel to see its ID. Save for later.

![]({{ "/public/img/tD19-app-id.png" | absolute_url }}){: .center-image}

## Packaging with command-line

You probably already know that in order to upload an extension to the store, it should be packaged as a zip file. We don’t want to do this step manually, though. We want to automate it. The **zip** command will help with this.

For example, in the terminal we can do:

```sh
$ zip -r my_file.zip .
```

And everything in the current directory will be compressed into a zip file.

We’ll adjust the command slightly in the next section.

## Bitbucket Pipelines

I’ve put together a basic Chrome extension starter project. [Download the code from Bitbucket](https://bitbucket.org/lewis_rodgers/chrome-extension-continuous-delivery/get/89e088a5137e.zip). Then, open `bitbucket-pipelines.yml`. This is the configuration file used by Bitbucket Pipelines. 

_For a detailed review of a Pipelines configuration file, see the [official Bitbucket documentation](https://confluence.atlassian.com/bitbucket/configure-bitbucket-pipelines-yml-792298910.html)._

{% gist dd39d48baf6cb041fd56a3ced01afcf1 %}

The pipeline is divided into two parts by branch name: develop and master. The intent is to call the **update** endpoint when code is pushed to the develop branch. But, call the **publish** endpoint when code is pushed to the master branch.

Have a look at the block of code under the **script** keyword. I bet you’ll recognize the last three lines. Let’s look at them in more detail.

## Package

The CWS Publish API requires extensions to be packaged as a zip file before upload. It’s a good idea to include only the required files needed for the extension to work. Exclude assets like unit tests, the node_modules folder, unrelated configuration files, etc.

Here’s a straightforward way to do this: Place extension–specific files into a subfolder called "app" and the zip command can be as simple as:

```sh
zip -r $FILE_NAME ./app
```

_This line prevents us from ever packaging the wrong extension files!_

## Authorize

During the build process, Pipelines requests an access token so it can make calls to the CWS Publish API. The access token lasts for 40 minutes before it needs to be refreshed. This is where a refresh token comes into play.

```sh
ACCESS_TOKEN=$(curl "https://accounts.google.com/o/oauth2/token" -d "client_id=$CLIENT_ID&client_secret=$CLIENT_SECRET&refresh_token=$REFRESH_TOKEN&grant_type=refresh_token&redirect_uri=urn:ietf:wg:oauth:2.0:oob" | jq -r '.access_token')
```

## Upload

With the access token in hand, an authorized request to the API can be made to update the extension.

```sh
curl \
-H "Authorization:Bearer $ACCESS_TOKEN" \
-H "x-goog-api-version:2" \
-X PUT \
-T $FILE_NAME \
-v "https://www.googleapis.com/upload/chromewebstore/v1.1/items/$APP_ID"
```

### Publish

To publish the extension we can use the following request:

```sh
curl \
-H "Authorization:Bearer $ACCESS_TOKEN" \
-H "x-goog-api-version:2" \
-H "Content-Length:0" \
-X POST \
-v "https://www.googleapis.com/chromewebstore/v1.1/items/$APP_ID/publish"
```

### Environment variables

To close the loop, we need to allow Bitbucket to continually interact with the CWS Publish API on our behalf. So, back in Bitbucket, add the client ID, client secret, refresh token, and app ID you’ve collected as environment variables to the **Pipelines settings** section. 

The app ID is publicly available, but you’ll want to secure the others so they aren’t revealed to prying eyes.

![]({{ "/public/img/tD19-environment-variables.png" | absolute_url }})

From here, your team can start pushing code to the develop and master branches. Pipelines will take care of packaging the extension, uploading it to the store, and publishing it.

## Conclusion

Updating and publishing extensions is a repetitive task and can be error prone when done manually. In this post, you learned about the Chrome Web Store Publish API and how it can be used to programmatically upload and publish extensions to the store. You also learned how to automate these tasks by combining the API with a continuous deployment tool.

If Bitbucket isn't your thing, try adapting what you’ve learned to work with your specific development stack.