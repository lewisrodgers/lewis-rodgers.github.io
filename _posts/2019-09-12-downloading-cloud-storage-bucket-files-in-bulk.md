---
layout: post
title: "Downloading Cloud Storage bucket files in bulk"
description: "With the Cloud Console you download files one at a time. Here are a couple options to do a bulk download with the gsutil tool."
date: 2019-09-12 00:00:00 -0400
tags: [GCP]
# image: "assets/posts/2019-09-12/featured.png"
# image-alt: ""
---
The Cloud Console doesn’t have a “download all” feature that lets you download every file in a Cloud Storage bucket all in one go. So, you’re left with downloading each file individually. That's tedious. 

But, there _is_ a way to do a bulk download,[^1] and it's with the gsutil tool.

First thing you should do is install gsutil. But if you don’t want to—or can’t—then the Cloud Shell is your alternative (everything you need is installed here and ready to go).

Ok, here’s the quick steps so you can get to work.

**Attention!** Don’t omit the period at the end of `gsutil cp -r gs://my-bucket .` (it’s part of the command).

## Option 1: Cloud Shell with gsutil

Activate[^2] Cloud Shell from the Cloud Console, then run these commands in the shell terminal:

```
mkdir my_folder
cd my_folder
gsutil cp -r gs://my-bucket .
zip -r all_files.zip my_folder
```

Download the `all_files.zip` file to your local machine and unzip it like you normally would.

{% include image.html name="download-file.png" caption="The vertical dots context menu contains the option to download." %}

Wondering about the zip? Similar to the Cloud Console, Cloud Shell only allows downloading one file at a time. To get around this limitation, package up multiple files into a single zip file. Then, download just the zip.

## Option 2: Locally with gsutil

Install and configure[^3] gsutil for your operating system, then run these commands in the terminal:

```
mkdir my_folder
cd my_folder
gsutil cp -r gs://my-bucket .
```

[^1]: ["Downloading objects \| Cloud Storage \| Google Cloud."](https://cloud.google.com/storage/docs/downloading-objects) Accessed 12 Sep. 2019.
[^2]: ["Quickstart \| Cloud Shell \| Google Cloud."](https://cloud.google.com/shell/docs/quickstart) 21 Aug. 2018. Accessed 12 Sep. 2019.
[^3]: ["Install gsutil \| Cloud Storage \| Google Cloud."](https://cloud.google.com/storage/docs/gsutil_install) 16 Aug. 2019. Accessed 12 Sep. 2019.