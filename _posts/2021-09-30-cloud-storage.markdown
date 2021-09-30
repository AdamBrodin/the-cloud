---
layout: post
title: "Cloud storage"
date: 2021-09-30 00:00:00 +0200
categories: cloud storage files
---

# Introduction

In this post I'm going to be going over the basics of storing files in the cloud. To do this I'm going to be using an [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs/), let's get started!

# Setting up

Start by creating a new storage account. Follow along as seen below:
![Storage account overview](https://i.imgur.com/jgRc2EL.png)

Enter a unique **storage account name** and select the default settings for this simple example. For a real-life production example you'll want to setup **redundancy** and go through all the other settings in detail.
![Storage account settings](https://i.imgur.com/IKufXW1.png)

Continue the steps via **review + create** and finally hit the **create** button. After a few minutes your new storage account is available.

# Talking to the storage account

You can communicate with your storage account through many different libraries and languages. I'm going to be demonstrating a basic example where you can **upload** an **image** and then retrieve the corresponding **URL** for it. Start by getting your **connection string**:

![Connection string](https://i.imgur.com/nCd61K1.png)
Copy that string and save it for later use.

# Project setup

Once that's done, open up your **.NET project** and add the **Azure Storage Blobs NuGet package**.

Quick-command: `dotnet add package Azure.Storage.Blobs`

For the sake of simplicity I'm just going to be accessing this **connection string** via a variable in the code. However this is something you **NEVER** should do in production or at all for that matter. Always store your sensitive keys/connection strings in environment variables. If you'd like to accomplish something similiar to do this you can use the following NuGet package: [https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json/](Microsoft.Extensions.Configuration.Json).

# Talking to the storage account

To establish a basic connection and retrieve a list of existing **blobs** for example is very simple.

```csharp
var connectionString = "MY_CONNECTION_STRING"
string containerName = "MY_CONTAINER_NAME";

BlobContainerClient container = new BlobContainerClient(connectionString, containerName);
```

Replace **MY_CONNECTION_STRING** with the connection string you copied earlier and **MY_CONTAINER_NAME** with a name of your choosing, this could for example be called "photos".

# Uploading a photo

First, add a photo of your choosing to the root folder of your project. My photo is going to be called "test_photo.png". Next add the following piece of code:

```csharp
// Makes sure the container exists/isn't null
container.CreateIfNotExists();

// Naming
string blobName = "test_photo";
string fileName = "test_photo.png";

/// Upload
BlobClient blobClient = container.GetBlobClient(blobName);
blobClient.Upload(fileName, true);
```

This code makes sure your container is valid, then uploads (in this case) a local photo as a **blob** with a set name.
If you want to retrieve an URL to the blob you've just created, you can easily do so with the following:

```csharp
// Prints https://{Storage Account Name}.blob.core.windows.net/{containerName}/{blobName}
Console.WriteLine(blobClient.Uri);
```

# Security

Security is as previously mentioned always a concern when dealing with anything cloud-related, and storage is definitely not an exception. Luckily **Azure** does a lot of the heavy lifting for you. This comes from its **automatic encrypting** when storing data, and **decrypting** when fetching data.

# Final words

With **Azure blob storage** you can easily, whether you're using large video-files or small text-files upload, download, backup and so much more with your data, all in the cloud. This greatly enhances the development experience as you won't have to think about manual backups or scaling.

# References

- <https://docs.microsoft.com/en-us/learn/paths/store-data-in-azure/>
- <https://cloudacademy.com/blog/how-does-azure-encrypt-data/>
