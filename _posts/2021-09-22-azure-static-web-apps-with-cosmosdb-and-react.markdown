---
layout: post
title: "Azure Static Web Apps with CosmosDB & React"
date: 2021-09-22 00:00:00 +0200
categories: cloud azure azure-static-web-apps react
---

# Introduction

In today's blogpost I'm going to, as mentioned in the previous post be going through how to actually implement and deploy a CosmosDB database to a webpage. I'm going to be doing this by using [Azure Static Web Apps](https://azure.microsoft.com/en-us/services/app-service/static/) to bundle together both the API and Frontend together.

# Getting started

It's really not that difficult accomplish this, start by creating your **Static Web App**:
![Azure Static Web Apps Pricing](https://i.ibb.co/2gK98GL/t34Aj38.png)

Be sure to select your deployment type of choice, I'm going to be using GitHub & GitHub Actions. Sign in to GitHub and authorize access to your account.
Once that's done, select the repository your project exists in.

Next, select your frontend framework of choice under build presets, I'm going to be using React.
The next step is very important, under app location, put the name of the folder which contains your frontend, in my case, it's named "frontend".
Do the same thing for "api location", enter the name of the folder which contains your api or **Azure Function**.

This should look something like this:
![Azure Static Web Apps Pricing](https://i.ibb.co/tmYLSW0/jWmIjtS.png)

When that's done you can hit create and Azure will automatically create a **CI/CD** workflow file in your repository to deploy your fullstack application to the cloud.
# Issues

Some issues can occur whilst trying to test and use an **Azure Static Web App** this way. Firstly, **MAKE SURE UR DB CONNECTION STRING IS SET!!!**. As you may recall from my last post, I put my **database connection string** in a seperate file called **local.settings.json**. This is something you manually have to do inside your **Static Web App's Configuration**. You can find this here:
![Azure Static Web Apps Pricing](https://i.ibb.co/jM88NXW/WDbh3Cx.png)

This was something I myself spent a long time trying to figure out as to why my database wasn't working.

Next up, **CORS**. In production you automatically gain access to your API, no modifications needed. However, when developing & testing locally you need to modify your connection to the **API** a bit.

If you're using React like me you can easily provide a "reverse-proxy connection" by adding this to **package.json**:

```json
  "proxy": "http://localhost:7071/"
```

And for your backend, put this in your **local.settings.json** file:

```json
  "Host": {
    "CORS": "http://localhost:3000"
  }
```

# Pricing

You might think a useful service like this comes with a fat price, but you'd be wrong. Other than the cost of maintaining the actual database & azure function, it's incredibly cheap regardless of project size. If you have a personal or really small project, it's even free! The pricing plan looks as follows:

![Azure Static Web Apps Pricing](https://i.ibb.co/MZ8fTdp/ejCdwxl.png)

# Conclusion

In this short post I've demonstrated how easy it is to deploy your **complete application** using **Azure Static Web Apps**. Something you may have noticed is I didn't actually provide any code for the frontend bits and that is to keep things a bit simpler. If you want to see the complete fullstack source-code you can find it here: [GitHub Repository](https://github.com/AdamBrodin/cosmos-todo-app)

# References

- <https://docs.microsoft.com/en-us/learn/modules/publish-static-web-app-api-preview-url/>
