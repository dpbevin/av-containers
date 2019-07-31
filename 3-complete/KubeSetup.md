# Setting up Kubernetes on Windows

It's important to note that Docker Deskop only supports Kubernetes when running Linux containers today.

1. Install recent version of [Docker Desktop](https://docs.docker.com/docker-for-windows/install/)
1. Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-windows)
1. Install the [Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/) (optional, but nice)
1. Right-click on Docker Desktop system tray icon, and select "Switch to Linux containers"
1. Right-click on Docker Desktop system tray icon, then select "Settings"
1. Navigate to the "Kubernetes" tab
1. Check the "Enable Kubernetes" option, then press "Apply"
1. Then wait...

NOTE: If you run into any difficulties when you're on the company network (SE Software), it's more than likely a problem with the Internet Proxy. The best advice is to work off the company network as much as possible.