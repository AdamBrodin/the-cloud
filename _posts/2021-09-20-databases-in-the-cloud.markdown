---
layout: post
title: "Databases in the cloud"
date: 2021-09-20 00:00:00 +0200
categories: cloud database azure azure-cosmos
---

# Introduction

In todays post I'll be going through how you can setup and utilize a database in the cloud via [Azure Cosmos](https://azure.microsoft.com/sv-se/services/cosmos-db/#overview).

# Setting it up

For this im going to be assuming you have some knowledge of Azure Functions, which I covered in my previous [blog post](https://adambrodin.github.io/the-cloud/cloud/azure/azure-functions/2021/09/14/azure-basics.html).

First and foremost you have to create the database. To keeps things as simple as possible I'm going to be doing all of this in VSCode. To make sure everything works smoothly you'll need these extensions:

- [Azure Account](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account)
- [Azure Functions](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
- [Azure Databases](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)

Start by opening the **command-palette** and **signing in**:
![Azure Sign In](https://i.imgur.com/FgHk9DS.png)

After you've followed the steps and are now signed in with your Azure account with a **valid subscription**, you're good to go!
Next up, creating the actual database, sample principle, open the **command-palette and follow the steps**:
![Azure Create Database](https://i.imgur.com/Ez29xUI.png)

Select Core (SQL), and follow the steps as prompted.

# The function (API)

As mentioned earlier, it is expected that you already have a C#/.NET function in which you can modify, whether that be locally or remotely, is up to you.
I'm going to modify my HttpTrigger class to make it look as follows:

```csharp
using System.IO;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Newtonsoft.Json;
using System.Collections.Generic;
using Microsoft.Azure.Cosmos;
using TodoApi.Models;

namespace TodoApi.Functions
{
    public class ApiHttpTrigger
    {
        private readonly CosmosClient _cosmosClient;
        public ApiHttpTrigger(CosmosClient client) => _cosmosClient = client;

        [FunctionName("PostTodo")]
        public async Task<IActionResult> PostTodo([HttpTrigger(AuthorizationLevel.Anonymous, "post", Route = "todo")] HttpRequest req)
        {
            string name = req.Query["name"];
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync().ConfigureAwait(false);
            dynamic data = JsonConvert.DeserializeObject(requestBody);
            name ??= data?.name;

            if (!string.IsNullOrEmpty(name))
            {
                await _cosmosClient.GetContainer("todo-db", "todos").CreateItemAsync(new TodoItem
                {
                    Id = System.Guid.NewGuid().ToString(),
                    Name = name
                }).ConfigureAwait(false);
            }

            string responseMessage = string.IsNullOrEmpty(name)
                ? "Invalid name."
                : $"Todo {name} created successfully";

            return new OkObjectResult(responseMessage);
        }

        [FunctionName("FetchAllTodos")]
        public async Task<IActionResult> FetchAllTodos([HttpTrigger(AuthorizationLevel.Anonymous, "get", Route = "todos")] HttpRequest req)
        {
            var iterator = _cosmosClient.GetContainer("todo-db", "todos").GetItemQueryIterator<TodoItem>(new QueryDefinition("SELECT * FROM todos"));
            var results = new List<TodoItem>();

            while (iterator.HasMoreResults)
            {
                var result = await iterator.ReadNextAsync().ConfigureAwait(false);
                results.AddRange(result.Resource);
            }

            return new OkObjectResult(results);
        }
    }
}
```

What this does is it creates **Restless API** endpoints for users to access, in my case, one **GET** and one **POST** method.

# Database communication

As you may have noticed I'm using a **CosmosClient** object in order to communicate with my database. To able to do this I first need to do two important things:

1. Dependency injection
   In order for the cosmosclient to work I need to inject it upon launch first.
   Doing this is quite simple and the code looks as follows:

```csharp
using System;
using Microsoft.Azure.Cosmos.Fluent;
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using TodoApi;

[assembly: FunctionsStartup(typeof(Startup))]
namespace TodoApi
{
    public class Startup : FunctionsStartup
    {
        private static readonly IConfigurationRoot Configuration = new ConfigurationBuilder()
            .SetBasePath(Environment.CurrentDirectory)
            .AddJsonFile("appsettings.json", true)
            .AddEnvironmentVariables()
            .Build();

        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddSingleton(_ =>
            {
                var connectionString = Configuration["CosmosDbConnectionString"];
                if (string.IsNullOrEmpty(connectionString))
                {
                    throw new InvalidOperationException("Invalid CosmosDBConnectionURL");
                }

                return new CosmosClientBuilder(connectionString).Build();
            });
        }
    }
}
```

2. Setup database connection string
   The client needs to know what database to talk to, and that is done by setting an **environment variable**.
   To retrieve your connection string you can very easily, within VSCode get it from here:
   ![Retrieve Database Connection String](https://i.imgur.com/tvJ3fui.png)

   After you've copied the connection string you have to put it in a specific file for environment variables, **local.settings.json** which can be found in the root of your **Azure Functions** folder.

   Paste in the string, to a key of your choice, which should look like this:

   ```json
   // local.settings.json
   "CosmosDbConnectionString": "AccountEndpoint=MY_URL;AccountKey=MY_KEY"
   ```

3. The model
   In order to actually read and write something to the database, you need (in general) a model. Mine looks very simple and represents one Todo-Task:

```csharp
using System.Collections.Generic;
using Newtonsoft.Json;

namespace TodoApi.Models
{
    public class TodoItem
    {
        [JsonProperty("id")]
        public string Id { get; set; }

        [JsonProperty("name")]
        public string Name { get; set; }

        [JsonProperty("completed")]
        public bool Completed { get; set; }

        [JsonProperty("tasks")]
        public List<Task> Tasks { get; set; }
    }
}
```

And that's it, your database should be fully **functional** and **accessible** both locally and remotely. Incase you missed something, or want the full source-code you can find it here: [Github Repository](https://github.com/AdamBrodin/cosmos-todo-app).

# Database details & pipeline

The database it self is a **CosmosDB NoSQL** database which stores information or **data** in a very accessible and readable **JSON-format**. In this example it consists of one "table" which are the todos themselves.

The complete pipeline when accessing the API now looks like this:\
**Client makes API call** -> **HTTP Trigger receives request** -> **Azure Function is executed** -> **Function connects to database using environment variables (db connection string)** -> **Function sends back response** -> **Function is done and no longer active/alive**

# Database migration

Something I wont be going into detail on in this blog post is **database migrations**. It can however be achieved in multiple different ways with various tools: [DB Migration Guide](https://docs.microsoft.com/en-us/azure/cosmos-db/import-data).

# Pricing

This example was purely for educational purposes but if you were to do something like this in a production, **real-life** scenario, money becomes very important.
The cost for a project such as this with a small user base (400 RU/S db) becomes relatively affordable, costing around **$20-25 USD per month** (with some additional costs when exceeding functions free limit of 400,000 GB/s & 1,000,000 executions).

And if you have a bigger project (100,000,000 executions/month & 100,000 RU/S) you're looking at around **$600-$700 USD per month** which in my opinion isn't too bad.

# Conclusion

In today's post I've gone through setting up a simple **Restful API** using **Azure Functions** utilizing a **CosmosDB** database. This is a cost-effective, fully-managed and in my opinion very handy way to access and use a database in a project in the cloud.

In my next blog post I'm going to be building upon this project to simulate a real-life scenario by creating a React webpage that talks to the **database API** I've created today.

# References

- <https://about-azure.com/working-with-cosmosclient-in-azure-functions/>
- <https://docs.microsoft.com/en-us/azure/azure-functions/functions-add-output-binding-cosmos-db-vs-code?pivots=programming-language-csharp&tabs=in-process#run-the-function-locally>
- <https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-cosmosdb-v2-input?tabs=csharp>
