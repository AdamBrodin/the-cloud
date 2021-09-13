---
layout: post
title: "Continuous Deployment with Docker"
date: 2021-09-13 00:00:00 +0200
categories: cloud continuous_deployment docker
---

# Introduction

In my previous [blog post](https://adambrodin.github.io/the-cloud/cloud/continuous_integration/github_actions/2021/09/07/continuous-integration-with-github-actions.html) I went through the basics of **continuous integration** and created a simple pipeline. In this post I'm going to be building upon that and for that I will be demonstrating continuous **deployment** or **CD**.

# What it is

**Continuous Deployment** is the natural next step in a **CI** pipeline. With **CD** you **automatically** deploy your built and tested project.
In my previous post I described a typical **CI** pipeline which looked like this:\
**Build your project -> run tests -> report back results**

But now with **CD** in our project we go all the way, from the codebase, all the way to the customer:\
**Build your project -> run tests -> report back results -> publish project to repository -> publish project to production**

# Benefits

So why would you want to use **continuous deployment** in your own project? Even though it can seem very tricky and daunting at first, the time you spend on setting it up, you'll save by automnating these tasks in the end.
Besides the large amount of time saved on repetitive yet trivial tasks, **CD** also comes with a bunch of other benefits. Mainly **CD** makes sure the finished product looks the same everytime you build. The built project uses standardized systems and versions of frameworks, all to minimize problems that could occur if you would manually deploy your application.

# A simple implementation

Let's look at a simple implementation of **CD**. For this I am going to be using a very simple project [Barista Project](https://github.com/AdamBrodin/the-barista-api-team-1) that brews coffee via the command-line. To be able to accurately test and develop this you'll probably need a version of [Docker](https://www.docker.com/) installed on your computer.

As I mentioned earlier, **CD** can be difficult to implement, but it is only as diffcult as you make it.
Let's start with the root of the pipeline, the **workflow file**:

```yml
name: Continuous Integration & Deployment
on:
push:
branches: - main
paths: - "**Dockerfile**"

jobs:
build:
runs-on: ubuntu-latest
steps: - name: Fetch codebase
uses: actions/checkout@v2
      - name: Setup Dotnet
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: "5.0"

      - name: Restore Dependencies
        run: dotnet restore BaristaApi

      - name: Build Api
        run: dotnet build -c Release --no-restore BaristaApi

      - name: Run Unit Tests
        run: dotnet test -c Release BaristaTests

      - uses: actions/upload-artifact@main
        with:
          name: webpack artifacts
          path: BaristaApi/

docker-build-and-push:
runs-on: ubuntu-latest
name: Docker Build & Push
needs: build
steps: - name: Fetch codebase
uses: actions/checkout@v1
      - name: Download built artifact
        uses: actions/download-artifact@main
        with:
          name: webpack artifacts
          path: BaristaApi

      - name: Build container image
        uses: docker/build-push-action@v1
        with:
          username: ${{github.actor}}
          password: ${{secrets.GITHUB_TOKEN}}
          registry: docker.pkg.github.com
          repository: adambrodin/the-barista-api-team-1/barista-api
          tag_with_sha: true
```

As you can see from the last post I've added some things.
The build jobs main purpose is now to do the following:\
**Restore/fetch the dependencies -> build the api project -> run the unit tests -> upload artifact (if successful)**

```yml
- name: Setup Dotnet
  uses: actions/setup-dotnet@v1
  with:
    dotnet-version: "5.0"

- name: Restore Dependencies
  run: dotnet restore BaristaApi

- name: Build Api
  run: dotnet build -c Release --no-restore BaristaApi

- name: Run Unit Tests
  run: dotnet test -c Release BaristaTests

- uses: actions/upload-artifact@main
  with:
    name: webpack artifacts
    path: BaristaApi/
```

Building upon that we have the docker part:

```yml
docker-build-and-push:
runs-on: ubuntu-latest
name: Docker Build & Push
needs: build
steps: - name: Fetch codebase
uses: actions/checkout@v1
      - name: Download built artifact
        uses: actions/download-artifact@main
        with:
          name: webpack artifacts
          path: BaristaApi

      - name: Build container image
        uses: docker/build-push-action@v1
        with:
          username: ${{github.actor}}
          password: ${{secrets.GITHUB_TOKEN}}
          registry: docker.pkg.github.com
          repository: adambrodin/the-barista-api-team-1/barista-api
          tag_with_sha: true
```

This part/job is responsible for actually fetching the built project from the previous step, creating a docker container image from that and finally publishing that to **GitHub Packages**. However, how does the workflow know how to build the Docker image? Well, that **magic** happens in the **Dockerfile**:

```Dockerfile
FROM mcr.microsoft.com/dotnet/runtime:5.0 AS base
WORKDIR /app

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["BaristaApi/BaristaApi.csproj", "."]
RUN dotnet restore "./BaristaApi.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "BaristaApi/BaristaApi.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "BaristaApi/BaristaApi.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "BaristaApi.dll"]
```

So what actually happens in the dockerfile? A simple way to put it is this:\
**Fetch .NET 5.0 images -> Move required files to their appropriate positions -> Restore -> Build -> and finally Publish the created image to the selected destination (GitHub Packages)**

That is the easiest way to explain things, there is no standardized way to create a Dockerfile, each framework and project will require its own research on how to put it together. This is the way for a simple .NET 5.0 project, but it will differ if you for example work on a React or Java project instead.

# The complete pipeline

The completed pipeline in detail looks a little something like this:\

**Commit code to branch -> Trigger workflow run -> Setup project dependencies -> Build project -> Run unit tests -> Upload artifact of built project -> Fetch artifact in deploy job -> Build docker-image from Dockerfile -> Publish docker-image to GitHub Packages**

And that's it.

The finalized output can be found as seen:
![Packages](https://i.ibb.co/PD5dPV2/pF0UOyh.png)\

And you can retrieve and run it locally with this URL:
![Image URL](https://i.ibb.co/PWnQHQs/98ONjU8.png)

# Conclusion

**Continuous deployment** is the next natural step to get a complete **CI/CD** pipeline and truly automize your project. With a slight starting curve it can be difficult at first but will greatly reward you in the future. Thank you for reading!

# References

- <https://www.youtube.com/watch?v=R5ppadIsGbA>
- <https://docs.docker.com/compose/gettingstarted/>

**Written by Adam Brodin**
