---
layout: post
title: "Azure basics"
date: 2021-09-15 00:00:00 +0200
categories: cloud azure azure-functions
---

# Introduction

In today's post I'm going to be going through the basics of serverless computing (FaaS) through [Microsoft Azure](https://portal.azure.com/).

# What is FaaS

**FaaS** or function-as-a-service is a cloud service which enables developers to execute code without any physical hardware. This is something most cloud-services provide and is one of the simpler levels of cloud computing. Common use cases include sending an email every x days to, for example remind users to read your newspaper.

# A simple example

FaaS is as mentioned one of the simpler means of cloud computing and therefore **can** be very simple to setup aswell. I'm going to be demonstrating how to build a simple addition calculator via **Azure Functions**.
After you've setup your account the steps are simple, first and foremost, create a **function app**:
![Create Function App](https://i.ibb.co/L9VJxdX/lapekg7.png)

When that's done you have to add **function**, for this example I'm going to be using a **HTTP Trigger**:
![HTTP Trigger](https://i.ibb.co/MV5skPs/reRsaSR.png)

What this means is the function and it's code is going to be executed everytime a **HTTP Request** is received to a certain url/endpoint.

Now all we have to do is add our code:

```csharp
#r "Newtonsoft.Json"

using System.Net;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Primitives;
using Newtonsoft.Json;

public static async Task<IActionResult> Run(HttpRequest req, ILogger log)
{
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    double sum = Convert.ToDouble(data?.firstNumber) + Convert.ToDouble(data?.secondNumber);

    return new OkObjectResult($"The sum of {data?.firstNumber} + {data?.secondNumber} is: {sum}");
}
```

What this simple code does is it takes **input** from the **HTTP Request**, in this case two numbers, which it then processes into an **output**. For the point of this exercise it simply performs some simple addition to produce an end sum of the two numbers, which it the returns to the origin of the **HTTP Request** as a **HTTP Response**.

# Testing

Testing this for yourself is very simple. You have to ways two main do it, first of all right through on the same page as the code is written. You can do this in the following steps:
![Testing Button](https://i.ibb.co/fM3BjnY/Pkl7Ksh.png")
![Input](https://i.ibb.co/qM0dh2G/UnY1DZe.png)

Which yields the corresponding output:
![Output](https://i.ibb.co/Xk9ZKh4/79NYhuk.png)

Another way to test this from your actual computer, is via **cURL** commands.
You can do this by fetching the **function URL** like so:
![Function URL](https://i.ibb.co/Fm9z6SF/8W5rE4o.png)

and then passing in your **request parameters** along an API authentication key which you can find here:
![App Keys](https://i.ibb.co/vPbWntw/L4z8uL5.png)

# Security

Something you **ALWAYS** have to consider when dealing with the cloud or really anything with computers these days is **security**. You can never be 100% secure against hackers but these are the top three things to maintain a secure enough environment for your data and functions:

- # Use authentication
  You should always require authentication via API keys when for exampel calling cloud functions. Not only does this reduce load and unwanted traffic but also reduces data breaches and other security risks.

- # Write secure code
  This one sounds very obvious but as mentioned you can never be 100% secure, however you still need to do your research on the various types of common breaches. What i'm specifically talking about here is injections. SQL injections, NoSQL injections and function runtime code injections are all very common ways to infiltrate your data and also execute unwanted and possibly malicious code.

- # Logging & monitoring
  Lastly, logging & monitoring. You should always monitor and log your cloud components. This makes it way easier to take action when the attack is there, and you can notice and hopefully resolve security issues before it's too late.

# Conclusion

This concludes this simple introduction to **FaaS** and cloud computing via simple functions like this one. Obviously this example was very simple but it still teaches you how **FaaS** can work in a small scale and how everything is linked together at the end, the complete **pipeline**.

# References

- <https://www.ibm.com/cloud/learn/faas>
- <https://docs.microsoft.com/en-us/learn/modules/choose-azure-service-to-integrate-and-automate-business-processes/>
- <https://docs.microsoft.com/en-us/learn/modules/create-serverless-logic-with-azure-functions/>
- <https://docs.microsoft.com/en-us/learn/modules/execute-azure-function-with-triggers/>
- <https://we45.com/blog/top-10-security-risks-in-serverless/>
