---
title: "Python with Docker Compose"
date: 2023-07-20T09:42:43+01:00
comments: false
tags:
  - python
  - docker
  - development
---


Developers hate it, how this one simple trick will lead to rapid python development

That’s it! What we’re going to do is setup a Python API running in docker-compose with hot reload enabled, meaning that any changes you make can be tested instantly once you save the file. If you’re familiar with Docker, you may be asking “How can the container know that I’m making a change to the code?”……Let me show you!

### Prerequisites

* Python 3.6+

* Docker

* Docker-Compose

First we need a python application to test this with. Let’s build a simple API server using FastAPI, a modern, lightweight web framework in python.

Let’s go ahead and make a directory somewhere on your computer

    $ mkdir compose-reload

Now let’s open that directory and create a python virtual environment.

    $ cd compose-reload
    $ python -m venv venv

Virtual environments are great for managing dependencies between projects, but that’s for another post. It’s been created and we need to activate it. Note that this command is going to be different on unix based systems and windows based systems.

**Unix:**

    $ source venv/bin/activate

**Windows:**

    $ venv\bin\activate.bat

Inside our virtual environment we will now want to install any dependencies, for us it’s going to be fastapi, the web framework, and uvicorn which is the server we are going to use to server our API.

    (venv)$ pip install fastapi uvicorn

Great! we’re ready to start coding.

go ahead and create a new file named main.py and add the below code

<iframe src="https://medium.com/media/84cd181b1eac1d3aaac12963a52a8bcb" frameborder=0></iframe>

that’s it! what we have here is an API at location /add that will take as input two separate integers and return a string indicating the sum. Let’s go ahead and run it using uvicorn

    (venv)$ uvicorn main:app --reload

Notice the reload, because that’s going to be important. You should see something like below in your terminal

    INFO:     Started server process [7916]
    INFO:     Waiting for application startup.
    INFO:     Application startup complete.

If you open your browser now and go to [http://localhost:8000/adder?a=1&b=3](http://localhost:8000/adder?a=1&b=3) you should be met with a white page displaying the following line.

    "The smu of 1 and 3 is 4"

It’s important to know what that reload means, so let’s go ahead and appease all of the OCD readers and correct the typo in the original file (replace smu with sum).

<iframe src="https://medium.com/media/a3f921ab2fa76e8ec2e517e18e0f0c3a" frameborder=0></iframe>

Once you save this file you should see an update in your terminal. Flagged by a WARNING log message.

    WARNING:  StatReload detected file change in 'main.py'. Reloading...
    INFO:     Started server process [7916]
    INFO:     Waiting for application startup.
    INFO:     Application startup complete.

Go back to the browser and reload

    "The sum of 1 and 3 is 4"

All is Correct with the world!

There’s one more thing we want to do before we move on to Docker. Right now we’re running uvicorn as the command to start the server, but we can actually run it from inside the Python file. This becomes pretty powerful because we can execute conditional code around the server such as checking using environment variables to configure the server information at runtime. Let’s make the following changes to our main.py file.

<iframe src="https://medium.com/media/95278b52936daf8de35e89e58bc631d9" frameborder=0></iframe>

With this change, instead of starting the server with uvicorn, you can just run main.py like any other python script. Notice the ENVIRONMENT variable, and how if it were set to anything other than “LOCAL” it would not start the server with the reload flag. This is important because the reload is cpu intensive and would not be a good thing to use in production.

Next up we need to dockerize this application so that we can run it in a docker-compose stack. Go ahead and create a new file named Dockerfile in the same directory.

<iframe src="https://medium.com/media/5d1501766097a95647560bdb5c29b0db" frameborder=0></iframe>

Finally, the last thing we need to do is create our docker-compose stack. This will be a new file named docker-compose.yaml

<iframe src="https://medium.com/media/21d4993bfe76f6a4133cad62c034eeef" frameborder=0></iframe>

We got a little fancy here and included a traefik service for routing traffic. If you’re not familiar with Traefik, it’s a popular reverse proxy used in a lot of modern stacks.

The important thing to notice and what makes this possible is the volume we defined on line 26.

    - ./:/app

What this means is that the current directory on your computer is going to be shared with the /app directory inside your api container. So, surprise surprise, when you make changes to your source code on your computer, the directory inside the container will be updated as well!

Here’s where the fun starts. let’s start the stack with

    (venv)$ docker-compose up

You will see your docker image being built and eventually starting, which should look similar to before. If you’re familiar with docker compose you may notice that we didn’t expose any ports for the api service, that’s because we’re actually using traefik to route traffic. So now if you go to your browser and type in: [http://api.localhost/add?a=5&b=9](http://api.localhost/adder?a=5&b=9)

    "The sum of 5 and 9 is 14"

There you have it, that’s how you can go about getting hot reload to work inside your docker compose stack to help you on your journey of rapid python development. Go ahead and try making some changes to your code and see if they are reflected in your browser. You could also try adding a database into your docker-compose stack….The world is your oyster!
