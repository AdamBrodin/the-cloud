---
layout: post
title: "Why you should use service buses"
date: 2021-09-28 00:00:00 +0200
categories: cloud azure azure-service-bus
---

# What is a service bus

A service bus is something that is used to decouple your application and its services from one another. In essence a service bus acts as a **message broker**, to send and receive messages across your whole application pipeline.

Common use cases include store/booking applications where it is essential that the clients requests (orders/purchases) gets processed from point A to point B securely, quickly and most importantly, without any risk of dataloss.

# The benefits

Service buses can bring a lot to the table that will benefit your applications.

# 1. Data integrity

As messages such as your order request as stored in the message and service bus, it'll remain intact all the way from the frontend until the order is completed. This can be as mentioned, crucial depending on your application and is especially important when dealing with financial applications or sensitive/urgent data.

# 2. Load balancing

Rather than talking directly to an API from the frontend, your service bus handles the work. Depending on the scale of your project this can heavily minimize your load and risks for requests not going through due to overloaded servers. Instead you can configure your service bus to queue messages and talk to your API for example in a timely manner, one at a time, in the order that they were received.

# 3. Decoupling

Because messages and services are **decoupled** or seperated from each other, the client does not need to wait for a response as opposed to direct contact with an API where everything freezes until the API response has been received. This leads to a better user-experience for the client (customer) and creates a "flow" through the whole process.

# Drawbacks

The main drawback of using a service bus comes down to single-point-of-failure. What I mean by this is in short terms; if the service bus fails, everything fails. Since everything will most likely be linked to towards your service bus, if something were to happen to it and it would seize to function as expected, all connected services would also raise issues. This is something that can be minimized or even avoided altogether though by utilizing solid monitoring and exception handling techniques.

# Conclusion

Service buses can be a great way to maintain a great user experience in your application by ensuring secure data delivery whilst also minimizing peak load towards your backend.
