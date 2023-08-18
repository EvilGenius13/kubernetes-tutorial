# Kubernetes Cluster Basic Setup Tutorial
Are you interested in learning about Kubernetes? In this tutorial, we'll walk you through the basic setup of a Kubernetes cluster. We'll start by deploying a web application that connects to a single database. Later, we'll explore scaling our application by replicating the web app. The best part is that Kubernetes has built-in load balancing capabilities, eliminating the need for third-party load balancers.

Throughout this tutorial, we'll cover various aspects of Kubernetes, including:

- Utilizing config maps and secrets to securely store sensitive information within the cluster.
- Leveraging deployments to host multiple containers within a single pod.
- Managing data storage within the cluster using persistent volume claims and persistent volumes.
- Exploring stateful sets for hosting multiple containers within a single pod.
- Using ingress to route traffic to our service and expose it to the outside world. *This is in the bonus section*

By the end of this tutorial, you'll have a solid understanding of the fundamental concepts of Kubernetes and how to set up a basic web application within a Kubernetes cluster.

If you have any specific questions or need further guidance, feel free to ask!

## Prerequisites
- [Docker](https://docs.docker.com/get-docker/) (Follow the instructions for your OS or install Docker Desktop)

## OPTIONAL - Take the flask app in this tutorial and build it yourself
Before you choose, let's talk about what the image even is. This is from a previous project we did in class. The image is a flask app that connects to a postgres database.  It's a simple app that allows you to create a backpack of pokemon. First, you create an account, and then login. You can add pokemon to your backpack, remove pokemon from your backpack, and view your backpack.  The image is already built and pushed to dockerhub.

If you want to use the image that is already made, you can skip this step as it's hard coded into the yml files. You can skip to [here](#install-minikube--kubectl)
If you want to build the image yourself start here: 
**Note: You will need a dockerhub account** - [Dockerhub](https://hub.docker.com/)

```bash
# Clone the repo
git clone https://github.com/EvilGenius13/PokeBackpackFinal.git
```
Create a `Dockerfile` in the **root** directory. (A dockerfile is a set of instructions that tells docker how to build your image)
A dockerfile does not have an extension.  It's just called `Dockerfile`. Right click or click new file and simply type `Dockerfile` and hit enter.
Fill your dockerfile with the following code:
```dockerfile
# Base image
FROM python:3.9

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file
COPY requirements.txt .

# Install the required dependencies
RUN pip install -r requirements.txt

# Copy the application code into the container
COPY . .

# Expose the port your Flask app will be running on
EXPOSE 5000

# Run the Flask app with Gunicorn - Then binding 0.0.0.0 allows any IP address to access the app instead of just localhost
CMD gunicorn app:app --bind 0.0.0.0:5000
```

Now that we have our dockerfile, we can build our image. Let's start by logging in.
```bash
docker login
```
Follow the prompts and enter your dockerhub username and password.

Run this command in the same folder as your dockerfile.
```bash
docker build -t <your dockerhub username>/your-app-name:latest .
```
The `.` at the end means build in the current directory, the `-t` flag is for tagging the image with a name and version
Also note, anything after the : is the version.  You can use whatever you want.  I just used latest. You can also use a version number like 1.0.0

Now you can push it to dockerhub:
```bash
docker push <your dockerhub username>/your-app-name:latest
```
If everything worked, you should be able to see your image in dockerhub. All you will have to do now is replace the image name in the yml files with your image name when we get to that step.

## Install Minikube & Kubectl
Follow the instructions for your OS
- [Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

## What is Minikube?
Minikube is essentially a Virtual Machine (VM) that runs Kubernetes. It's important to start it with the docker driver.  This means it will create a VM using docker instead of virtualbox which is a standalone VM system. 

## What is Kubectl?
Kubectl is the command line tool that allows you to interact with your Kubernetes cluster.  You can use it to create deployments, services, pods, and more.  You can also use it to get information about your cluster.

## Start Minikube
It's important to start it with the docker driver.  This means it will create a VM using docker instead of virtualbox which is a standalone VM system.
Let's start by making docker the default driver for minikube.
```bash
minikube config set driver docker
```
Now we can start minikube. You can also specify the amount of memory you want to allocate to the VM. I recommend at least 4GB.
```bash
minikube start --driver=docker
minikube start --driver=docker --memory=4096
```
You can now check the status of your cluster with the following command.
```bash
minikube status
```
You should see something like this:
```bash
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

You can also get the IP address of your cluster with the following command.
```bash
minikube ip
# You'll need this later to access your application locally
```

## YAML or YML Files
Now that we have our cluster up and running, we can start deploying our application.  Kubernetes needs instructions to deploy applications, and that's where YAML files come in. Think of these files as "recipe cards" that tell Kubernetes exactly how to set up and run your app. Here’s what each of our "recipe cards" does:

ConfigMap (pokeapp-configmap.yml): Holds configuration data and settings. Imagine it as app settings that are external from the app itself.

Secret (pokeapp-secret.yml): Safely stores sensitive data, like passwords. Picture it as a secure vault for app secrets.

PokeApp Deployment (pokeapp-deployment.yml): Tells Kubernetes how to run your app, like which image to use or how many replicas to create.

Postgres Deployment (postgres-deployment.yml): Just like the PokeApp Deployment but specifically for running the Postgres database.

Ingress (pokeapp-ingress.yml): Think of this as the front door. It manages external access to the services, typically HTTP.

So, how does Kubernetes distinguish between these "recipe cards"? By using labels! In each file, there's a kind label. This kind acts like the title of the recipe card, telling Kubernetes the purpose of each file. For instance, if it reads kind: Deployment, Kubernetes knows it's about deploying an app or service.

Let's dive deeper into what each of these "recipe cards" entails.

### ConfigMap
Imagine the ConfigMap as a customizable settings panel for your application. Instead of adjusting the code of your app every time you want to change a setting, you can just tweak the ConfigMap. It's particularly handy for managing items like database connections or any dynamic variables. In more advanced setups, ConfigMap lets you tailor settings for different environments, like development, staging, or production. For our purposes, think of it as a flexible tool we're introducing now, even if we're just scratching its surface. Our reference file for this is `pokeapp-configmap.yml`.

### Secret
Imagine a 'Secret' in Kubernetes as a special locked box, designed to hold your sensitive details like passwords or API keys. Now, while this box isn't impossible to open, it does add a layer of obfuscation.

- What it is: A 'Secret' is a way to tuck away sensitive data.

- Encoding, not Encryption: Remember, the contents of this box are encoded using Base64 – they aren't encrypted. It's like scrambling up the letters of a word, rather than using a cipher. So while they're not in plain sight, someone determined could still make out what's inside.

- Why use it?: It's always a good idea to keep passwords and sensitive data out of your main application code. While the Base64 method isn't the most secure, it's a straightforward starting point in Kubernetes.

- Filename: Our specific secret file is mongo-secret.yml.

- Contents: We're storing three bits of info here: a username, a password, and a secret word.

- Creating a Secret: Kubernetes requires your secrets to be in Base64 format. If you're curious about how a word looks when encoded in Base64, try these commands:

```bash
echo -n "Ash" | base64
echo -n "Ketchum" | base64
echo -n "pokemon" | base64
```
This technique is how we get the encoded strings you'll see inside our secret file.

### Deployment & Service
Deployments are like the blueprint of your application. It tells Kubernetes what your application looks like and how to run it.
Services are like an address book. It tells Kubernetes how other services or users can reach your application.
You can write these instructions in separate files or combine them into one. For our purposes, we'll keep them together in one file.
Our Service & Deployments are `postgres-deployment.yml` and `pokeapp-deployment.yml`
Postgres is an open source SQL database. Pokeapp is our flask based application.
In our case, the Service and Deployment is separated by '---' in the same file.

### Names & Labels & Ports
Name: Think of this as a unique nickname for your app in Kubernetes. It doesn't have to match the name of the Docker image. You'll use this "nickname" when you want to manage or refer to the app in Kubernetes commands.

Image: This is your application packed in a container, ready to be run. Kubernetes will fetch this from Docker Hub or other container registries.

Label: Labels are like tags or sticky notes you put on deployments. If you have many apps running, labels help you quickly find and manage groups of related apps.

Port (Container Port): Imagine your application inside the container has a door; this is that door number. In our case, it's 5000 for the pokeapp.

Target Port: When Kubernetes wants to send traffic into your container's door (port), it uses this. Think of it like a funnel directing traffic. It often matches the container port.

NodePort: It's a special type of port in Kubernetes. If you want to access your app from outside Kubernetes, this port does the job. It's always between 30000-32767. For our pokeapp, we are using 30100.

An Example:
Let's say someone wants to visit your app. They knock on door number 30100 (NodePort). Kubernetes then directs (or funnels) them to door number 5000 (Target Port & Container Port) of the pokeapp container.

### Deploying to Kubernetes
We need to give the cluster our config map and secret before we can deploy our application. This is because postgres and pokeapp require that information. We can do this with the following commands.
```bash
kubectl apply -f mongo-config.yml
kubectl apply -f mongo-secret.yml
```
Now we can deploy our application. Do the mongo dabatase first as the webapp depends on it.
```bash
kubectl apply -f mongo.yml
kubectl apply -f webapp.yml
```
You can check the status of your deployments with the following command.
```bash
kubectl get deployments
```
You can check the status of your pods with the following command.
```bash
kubectl get pods
```
### Add more replicas
Wait, we're only running 1 database, and 1 webapp. We should scale our webapp for more traffic. We can do this with the following command.
```bash
kubectl scale deployment webapp --replicas=3
```
Alternatively, you can edit the webapp-deployment.yml file and change the replicas to 3 and then run `kubectl apply -f webapp-deployment.yml`

Now run `kubectl get pods` again and you should see 3 pods instead of 1. 

### Looking at the data and logs
You can look at the data in your database with the following command.


### Ingress
```bash
minikube addons enable ingress
```
Check the install
```bash
kubectl get pods --all-namespaces
```
