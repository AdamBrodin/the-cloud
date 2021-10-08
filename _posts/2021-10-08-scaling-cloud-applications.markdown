---
layout: post
title: "Scaling cloud applications"
date: 2021-10-08 00:00:00 +0200
categories: cloud scaling
---

# Introduction

In todays blog-post i'm going to be going over how you stress-test your **cloud application** to see how well it **scales**. This is something that can overlooked when developing and testing in a development environment, only to crash your services due to the load being too much, so it's good to test the performance of your application before deploying to production.

In my case i'm going to be **stress-testing** an **Azure Function** that lies in an **Azure Static Web App**, more details on this can be seen in my [previous blogpost](https://adambrodin.github.io/the-cloud/cloud/azure/azure-static-web-apps/react/2021/09/21/azure-static-web-apps-with-cosmosdb-and-react.html).

Let's start by comparing different scaling methods.

# Horizontal Scaling

Horizontal scaling is a scaling method in which the **computing power** increases by adding **additional** hardware or servers. This is done without altering or improving your current hardware, the hardware only increase in **quantity, not quality**.

# Vertical Scaling

Vertical scaling on the other hand does the opposite, it keeps and upgrades the current hardware infrastructure. With vertical scaling, you build upon your servers, add more ram, better or more CPU's etc.

# Drawbacks

Regardless of what scaling strategy you pick there are gonna be some drawbacks. With **horizontal scaling** the major drawbacks are: **data consistency** due to the data existing in many different servers at once and higher costs as a result of more development time and the cost of servers are more expensive when it comes to price vs performance.

And with **vertical scaling** some major drawbacks include the possibility of additional downtime due to the low quantity or servers running at once. Furthermore there is an increased risk of data-loss or major downtime due to hardware failure.

# Stress-testing

Now that you've gotten some insights in how to structure your scaling, let's do some practical testing. I'm going to be using a [**free load-testing service**](https://loader.io/).

## Setting up

Start by creating an account at **loader.io** and then add your the URL for your website. Next we have to verify the **loader.io** connection inside our **azure function**. We can do this easily by first creating a **proxies.json file** in the root directory of the function app:
![Proxies Json](https://i.imgur.com/eivAEEN.png)

And adding the following piece of code:

```json
{
  "$schema": "http://json.schemastore.org/proxies",
  "proxies": {
    "loaderio-verifier": {
      "matchCondition": {
        "methods": ["GET"],
        "route": "/loaderio-verification-token"
      },
      "responseOverrides": {
        "response.body": "loaderio-verification-token",
        "response.headers.Content-Type": "text/plain"
      }
    }
  }
}
```

Replace "loaderio-verification-token" with the token found on the **loader.io** website:
![Verification token](https://i.imgur.com/Ek2Cc6G.png)

Also add this to your **.csproj** project file:

```xml
<None Update="proxies.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
</None>
```

It is also very recommended that you use environment keys when dealing with api keys/token and this can be done easily with the following:

- Add the key to **local.settings.json**
  ```json
  "loaderio_token": "loaderio-verification-token"
  ```
- Change **proxies.json** to access the key:
  ```json
  "route": "/%loaderio_token%"
  "response.body": "%loaderio_token%",
  ```

And that's it, your loader.io verification should be all set up. You can test this out by hitting "verify" on the **loader.io** website. After that all you have to do is start the tests, let them run for a few minutes and analyze the results by viewing the metrics, or creating a custom query as mentioned in my [previous post](https://adambrodin.github.io/the-cloud/cloud/monitoring/2021/10/05/monitoring-of-cloud-applications.html)

# Conclusion

Load-testing is an easy, essential and very important step when developing a production application to identify performance issues and make sure your deployment and production environment works as smoothly as possible no matter the amount of traffic.

# References

- <https://touchstonesecurity.com/horizontal-vs-vertical-scaling-what-you-need-to-know/>
- <https://www.rootstrap.com/blog/horizontal-vs-vertical-scaling/>
- <https://mikhail.io/2019/07/load-testing-azure-functions-with-loaderio/>
- <https://blog.danielreis.dev/using-function-proxies-with-azure-static-web-apps>
