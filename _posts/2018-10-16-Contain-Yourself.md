---
layout: post
title: Contain Yourself!
tags: [docker, elif]
---

![xkcd](https://imgs.xkcd.com/comics/containers.png)

## What is Docker?
Docker packages everything you need to run an application into "containers".  These containers are portable, meaning they can be shared with others who want to recreate your environment and run your app. Docker makes your work more:
* Flexible
* Lightweight
* Interchangeable
* Portable
* Scalable
* Stackable
With docker, continuous integration and continuous development is seamless.  Dockerized applications don't have system dependencies, updates can be pushed to any part of a distributed application, and resource density can be optimized.

## Docker terms
* <b>Image:</b> A blueprint for what you want to build
* <b>Container:</b> An instantiation of an image.  If the image is the blueprint, the containter is the building. You can have multiple containers built with the same image. 
* <b>Dockerfile:</b> A text document that contains instructions for creating a image.  These instructions are commands that a user could call in the command line to assemble an image.
* <b>Layer:</b> A modification to an existing image, as per instructions in the Dockerfile.
    
## Let's code a docker container!
First, make sure that you have docker [installed]('https://docs.docker.com/docker-for-mac/install/').

### Define a container with `Dockerfile`.
As mentioned above, we need a `Dockerfile` to define what does on in the environment inside the container.  This is the blueprint for your virtual machine. It may look something like this:

```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
This Dockerfile starts with an image that contains a Python runtime. It provides version 2.7 in a slim configuration, meaning it contains a minimal number of Python packages.

Next, it establishes a WORKDIR (working directory) named /app and ADDs the current working directory to it.

After adding the script to the image, we need to pip install the packages required to run our appliation.  

Then, we need to specify the localport in which our app is run.  In this case, it is port 80.

Next, it sets the environment variable NAME to World.  

And finally, the Dockerfile specifies the command to run when the image is run. CMD accepts a command and a list of arguments to pass to the command. This image executes the Python interpreter, passing it app.py.

### requirements.txt
This text file includes a list of all the dependencies required to make your application run.  For example:
```
Flask
Redis
```
`pip install -r requirements.txt` will install Flask and Redis libraries to your container.

### app.py
Where the code for your application lives!

```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

## Let's build the app!
So now we have 3 files in our directory. `Dockerfile`, `requirements.txt` and `app.py`.

To build the docker image, we can run the build command.
`docker build -t friendlyhello .`

We can see if our docker image built successfully using `docker image ls`.

## Let's run the app!
To run the app, we can map your machine’s port 4000 to the container’s published port 80 using -p:

`docker run -p 4000:80 friendlyhello`

You should see a message that Python is serving your app at http://0.0.0.0:80. But that message is coming from inside the container, which doesn’t know you mapped port 80 of that container to 4000, making the correct URL http://localhost:4000.

Go to that URL in a web browser to see the display content served up on a web page.

We can also run the app in the background using detached mode `-d`.

`docker run -d -p 4000:80 friendlyhello`

You'll get the long container ID for your app before getting redurected to your terminal. Your container is running in the background. You can also see the abbreviated container ID with `docker container ls` (and both work interchangeably when running commands):

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
```

Now, if you go to `http://localhost:4000`, you can see your app!

## Stop the app
`docker container stop 1fa4ab2cf395`