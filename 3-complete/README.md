# Orchestrate ASP.NET Core and MS SQL Server

## Prerequisites
Before you begin, you will need:
* Windows 10 (1803 or higher)
* [Docker Deskop](https://docs.docker.com/docker-for-windows/install/)
* Kubernetes (see [Setting up Kubernetes](KubeSetup.md))
* ASP.NET Core SDK 2.2
* An editor such as VS Code, Visual Studio 2019, Notepad, or even VIM.

## Overview

In this example, we will:
1. Modify the sample MVC application to connect to a database.
1. Publish the ASP.NET Core application to a folder
1. Build an image with the application
1. Orchestrate the containers

---

### Modify the MVC application

Building on top of the ASP.NET Core application from stage 2, modify to include "code-first" Entity Framework Core.

The code in this sample has already been updated for you. See this guide for step-by-step instructions: https://docs.microsoft.com/en-us/ef/core/get-started/aspnetcore/new-db?tabs=visual-studio

In the [Startup.cs](ProductApi/Startup.cs) file you'll see that the connection string is formed from three separate environment variables

```
var server = this.Configuration["MY_SERVER"];
var user = this.Configuration["MY_USER"];
var pass = this.Configuration["MY_PASSWORD"];

var connection = $"Server={server};Database=MyProducts;User ID={user};Password={pass};Trusted_Connection=False;ConnectRetryCount=0";

```

### Publish the ASP.NET Core application

Open a Powershell (or command) prompt to this directory and run the command:

```
PS> dotnet publish -c Release -o .\..\Release
```

You should see a "Release" folder created in the current directory.


### Build a container for the application

Inspect the [dockerfile](dockerfile) in this directory and note that is is virtually identical to the one we used before. Only the ```ENV``` command has been removed.

Now build the container as we did before.

This time, note that we provide the version of ```v2```.

```
PS> docker build -t techtopics/productapi:v2 .
```

### Orchestrate the containers

There are three steps to setup the system:
1. Configure a password to be used for SQL Server access
1. Provision the MS SQL Server container
1. Provision the web container

#### Configuring the password

The web application and SQL Server both need a user password. In this sample, we're going to use the "sa" user account because the password for this can be specified during container startup.

**WARNING: You shouldn't use the "sa" account in production!**

Instead of just setting the password in plain text, we're going to store the password in a Kubernetes **Secret** so that it can be protected.

Kubernetes secrets are stored in Base64 encoded format. So let's take a imaginary password "PassW0rd!23" and encoded it using Powershell:

```
PS> $pwd  = [System.Text.Encoding]::UTF8.GetBytes("PassW0rd!23")
PS> [System.Convert]::ToBase64String($pwd)
```

Save the encoded password to the [0-secret.yaml](0-secret.yaml) file on line 6. Inspect the file and note that we're storing the name of this secret as "sqlserver-secret". We can use this name to refer to the password elsewhere.

NOTE: YAML files are used to configure deployments within Kubernetes.

Now deploy the secret using the kubectl command.

```
PS> kubectl apply -f .\0-secret.yaml
```

You should see the response:
> secret "sqlserver-secret" configured

#### Provisioning the SQL Server

Next we provision the SQL Server instance. We need a *Deployment* and a *Service*. The Service allows us to expose the SQL Server port (1433) to other containers within the cluster.

This container is provisioned using the [1-sql.yaml](1-sql.yaml) file.

Some key items from the file:
* Line 4: We identify this service as "productdb-service". We can use this in other yaml files.
* Lines 9-12: We expose the SQL Server port (1433) to the cluster
* Line 32: We want to start the MS SQL Server image
* Lines 38-42: We configure the "sa" user password

We run a familiar command to provision the container and Service:

```
PS> kubectl apply -f .\1-sql.yaml
```

You should get two messages to say that the service and deployment have been provisioned.

We can check this by running two more kubectl commands: ```kubectl get services``` and ```kubectl get deployments```.

```
PS> kubectl get services                                                                       NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
kubernetes          ClusterIP   10.96.0.1      <none>        443/TCP    8d
productdb-service   ClusterIP   10.101.15.81   <none>        1433/TCP   1m

PS> kubectl get deployments                                                                    NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
productdb-deployment   1         1         1            1           1m
```

#### Provisioning the web service

We provision the web service container in a similar manner to the SQL Server deployment. This container is provisioned using the [2-webapp.yaml](2-webapp.yaml) file.

Some key points from this yaml file are:
* Line 8: We expose this service as a **LoadBalancer** so that it's available outside of the cluster (i.e. to the host)
* Line 10: The web service will be available on host port 8123
* Line 33: Specify our web service image
* Line 39: Contains the machine name of the SQL Server container (used for the connection string)
* Lines 42-46: Specify the "sa" user password

Then we run the ```kubectl apply``` command to provision the deployment and service.

```
PS> kubectl apply -f .\2-webapp.yaml
```

Verify the correct provisioning using the ```kubectl get services``` and ```kubectl get deployments``` commands.

You should now be able to open a browser to the web page: http://localhost:8123/Products and add, edit, and delete products.

#### BONUS: Scaling the web service

Running a single container for the web service isn't something you'd typically do in a production system.

Scaling up the number of web containers is a simple as running the command

```
PS> kubectl scale --replicas=2 deployment/productapi
```

You can then check the appropriate number of containers are running by executing the command

```
PS> kubectl get deployments
```

## Cleaning Up

To delete the resources we provisioning here, run the commands:

```
PS> kubectl delete -f 2-webapp.yaml
PS> kubectl delete -f 1-sql.yaml
PS> kubectl delete -f 0-secret.yaml
```
