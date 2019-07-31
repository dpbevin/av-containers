# ASP.NET Core Web App in a Container

## Prerequisites
Before you begin, you will need:
* Windows 10 (1803 or higher)
* [Docker Deskop](https://docs.docker.com/docker-for-windows/install/)
* ASP.NET Core SDK 2.2
* An editor such as VS Code, Visual Studio 2019, Notepad, or even VIM.

## Overview

In this example, we will:
1. Modify the sample MVC application to show information from an environment variable.
1. Publish the ASP.NET Core application to a folder
1. Build an image with the application
1. Run the image as a container

---

### Modify the sample MVC application

There are only two modifications to the sample MVC application.

Modify the ```Index()``` method in [HomeController.cs](ProductApi/Controllers/HomeController.cs) to store a message to the ```ViewData```.
NOTE: You also need to create a constructor and save an instance of ```IConfiguration```.

Next, modify the ```<h`>``` element in [Index.cshtml](ProductApi/Views/Home/Index.cshtml) to show ```@ViewData["Message"]``` instead of "Welcome".

Save files.

### Publish the ASP.NET Core application

Open a Powershell (or command) prompt to this directory and run the command:

```
PS> dotnet publish -c Release -o .\..\Release
```

You should see a "Release" folder created in the current directory.

### Build a container for the application

Inspect the [dockerfile](dockerfile) in this directory and note that:
1. The base image is the standard ASP.NET Core 2.2 Runtime image
1. The "Release" folder is being copied from the host machine into the "app" folder of the image
1. The environment variable is being set to show the value in the browser
1. The container will be started by running the ```dotnet``` command and pointing to our application

Now build the container as we did before.

This time, note that we provide the version of ```v1```.

```
PS> docker build -t techtopics/productapi:v1 .
```

### Running the container

When we run the container this time, we're going to:
* Automatically delete the container when it exits by using the ```--rm``` argument
* Run the container interactively so we can see the output by using the ``--it`` argument
* Map port 80 from the container to port 5050 on the host machine by using the ```-p``` argument

```
PS> docker run -it --rm -p 5050:80 techtopics/productapi:v1
```

If everything works correctly, you should be able to view the web server at [http://localhost:5050/](http://localhost:5050/)


#### Stopping the container

To stop the container from the interactive session, simply type ```Ctrl-C```.

#### Checking the container has been deleted

Running the following command should show that the productapi container has been deleted.
```
PS> docker ps -a
```

---

## Next Steps...

Try removign the ```ENV``` line from the dockerfile, then using the ```--env``` argument to ```docker run``` instead.

See https://docs.docker.com/engine/reference/commandline/run/#set-environment-variables--e---env---env-file

Providing environment variables can be done from various locations depending on needs.