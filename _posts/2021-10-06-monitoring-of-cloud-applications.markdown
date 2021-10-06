---
layout: post
title: "Monitoring of cloud applications"
date: 2021-10-06 00:00:00 +0200
categories: cloud monitoring
---

# Introduction

In today's post i'm going to be looking at monitoring a cloud application in order to achieve better insights into performance, load, security and much more valuable information for my project. Specifically an existing Azure Static Web App from a previous [blogpost](https://adambrodin.github.io/the-cloud/cloud/azure/azure-static-web-apps/react/2021/09/21/azure-static-web-apps-with-cosmosdb-and-react.html).

# Enabling monitoring

If application insights is not enabled by default, you can very easily enable it for your **Static Web App** by doing the following:
![Application Insights](https://i.imgur.com/sLB5jIv.png)

And that's all it takes, your application now automatically tracks things such as **response time, availability and amount of requests**.

# Querying data

To be able to read data, both manually and through queries you have to open up your newly created **Application Insights** resource. From the start-page of the portal:
![Portal startpage](https://i.imgur.com/eNVDXIJ.png)

The newly created resource:
![Insights Resource](https://i.imgur.com/6uZgLFW.png)

From here on you can find loads of data, configuration settings and metrics. To able to query your metrics you have to open up the "logs" page:
![Logs page](https://i.imgur.com/ZMQtZTT.png)

Here you can find **a lot** of different **pre-defined** queries written using the **Kusto Query Language** which is similiar to SQL. For example, if you wanted to see the load (requests) for your webpage for the past day as to identify highs and lows for your traffic, you could write it very simply as follows:

```sql
requests
| summarize totalCount=sum(itemCount) by bin(timestamp, 30m)
| render timechart
```

Or maybe you want to identify issues with your website, you can query the most common exceptions that cause a page request to fail:

```sql
requests
| where timestamp > ago(1h) and success == false
| join kind= inner (
exceptions
| where timestamp > ago(1h)
) on operation_Id
| project exceptionType = type, failedMethod = method, requestName = name, requestDuration = duration
```

As you can see, there is a lot of potential with **Kusto Queries** and you can get insights for pretty much anything. Insights come pre-defined with a plethera of tables (data) that you can access through your queries, and they can be found here:
![Tables](https://i.imgur.com/l1eYupX.png)

Both the **requests** and **exceptions** table as I used in my example queries lie here.

# Security benefits

Implementing analytics and insights into your application can be a very efficient tool in order to identify security flaws and attacks. For example, a common attack, **Denial of Service attack** could be detected early on via **alerts** that get triggered once the requests for that hour exceed a certain threshold. That would lead to a higher chance of the website staying alive and action could be taken as soon as an attack was detected. In the end, this would mean more uptime = more money.

# Conclusion

In this post I've gone through the basics of monitoring a cloud application via [**Azure Application Insights**](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview) and demonstrated how easy it can be to setup and utilizie to maximize uptime, and quickly analyze problems.

# References

- <https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview>
- <https://docs.microsoft.com/en-us/azure/static-web-apps/monitor>
