---
layout: post
title: "Continuous Integration with GitHub Actions"
date: 2021-09-08 01:07:53 +0200
categories: cloud continuous_integration github_actions
---

# Continuous Integration

You might've heard terms like **CI/CD** or **continuous integration/continuous deployment** before, and wondered, what is it? To put things simply, **CI** is a means of automating tasks in your project, CI could for instance be triggered everytime you commit to a branch.

# Why you need it

Why so many developers tend to use **continuous integration** in their projects regardless of size is no mystery. CI can and will save you both time, money and countless headaches. Jokes aside, think of CI like your servant that does the essential and boring parts of your project for you.

# What it can do

In a real world, practical example, **CI** usually does a variety of things. Most commonly a simple **CI** pipeline goes a little something like this:

**Build your project -> run tests -> report back results**

Something I'm not going to be covering in this post is **CD** or **continuous deployment** which is essentially what happens after the results. If your build fails, you'll get alerted with what the issue is so you can resolve the situation quickly. Although this is a common workflow when using **CI**, the limits of what you can do with it lies in the clouds.

# An actual example

I have a [project](https://GitHub.com/AdamBrodin/spacepark-spacepark-group3) where I want to run a certain action everytime I commit to it. How would I do this?
To execute your action you need a **workflow** file. A workflow file looks tricky at first but once you spend a little time with it (and its indentations!!!) you'll find it rather easy.
You can execute your **CI** operations & actions anywhere and anyhow you'd like but in this example im going to be using **GitHub Actions**.

# GitHub Actions

With GitHub Actions it's really not difficult to automate trivial tasks. First of all we have to create a special directory in our repository to let GitHub know where our **workflow file** is. This directory should be called **.GitHub/workflows/YOUR_WORKFLOW_NAME**.
Once you've created that directory, create your **workflow file** in that directory and name it anything you'd like.

To explain how to structure your **workflow file** im going to show you the finished file first.

# my_workflow.yml

```yml
name: on_commit
on: push
jobs:
  print_job:
    runs-on: ubuntu-latest
    steps:
      - name: print_hello_world
        run: echo "Hello World!"
```

Let's go through each variable one by one.

```yml
# the naming of your workflow
name: hello-world
```

```yml
# the trigger for the workflow and its jobs (actions) to be executed
# in this case it's set to "push" which means on every push/commit to the repository
on: push
```

```yml
# this defines which jobs the workflow contains and their names
jobs:
  print_job:
```

```yml
# what type of machine the job is run on (or commonly, in simpler terms, what computer/system, for example linux or windows)
runs-on: ubuntu-latest
```

```yml
# this defines the steps the workflow contains
steps:
  - name: print_hello_world # the name of the step
    run: echo "Hello World!" # the action/task the step performs
```

And that's it! If you followed along this far on your own you have successfully integrated a very simple version of **continuous integration** in your own project.
After you've commit something you can head over to the **"actions"** tab on your repository and you should see something like this:
![Workflow Run](https://i.ibb.co/z8LQLN6/workflow-run.png)
And if you click on your **workflow run** you can see the status (or finished result) of your job, which looks something like this:
![Job output](https://i.ibb.co/8Kr77CL/job-output.png)

Thank you for reading this post, this was just a very simple example of **CI** but I still hope you have learned something about **continuous integration**, why to use it and most importantly how to implement it.

**Written by Adam Brodin**
