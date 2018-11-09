## Learning Kubernetes

This is just a simple demonstration to get basic understanding how kubernetes works while working step by step.

## Requirements

- You need to have [docker](https://www.docker.com/) installed for your OS
- [minikube](https://github.com/kubernetes/minikube) installed for running locally
- [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) installed.

## Simple conepts before we start

#### What is docker

Docker is platform for packaging, distribution and running applications. It allows you to package your application together with its whole environment. This can be either a few libraries that the app requires or even all the files that are usually available on the filesystem of an installed operating system. Docker makes it possible to transfer this package to a central repository from which it can then be transferred to any computer running Docker and executed there

Three main concepts in Docker comprise this scenario:
- **Images** :— A Docker based container image is something you package your application and its environment into. It contains the filesystem that will be available to the application and other metadata, such as the path to the executable that should be executed when the image is run.
- **Registries** :- A Docker Registry is a repository that stores your Docker images and facilitates easy sharing of those images between different people and computers. When you build your image, you can either run it on the computer you’ve built it on, or you can push (upload) the image to a registry and then pull (download) it on another computer and run it there. Certain registries are pub- lic, allowing anyone to pull images from it, while others are private, only accessi- ble to certain people or machines.
- **Containers** :- A Docker-based container is a regular Linux container created from a Docker-based container image. A running container is a process running on the host running Docker, but it’s completely isolated from both the host and all other processes running on it. The process is also resource-constrained, meaning it can only access and use the amount of resources (CPU, RAM, and so on) that are allocated to it.

## Learning while working

You first need to create a container image. We will use docker for that. We are creating a simple web server to see how kubernetes works.

- create a file `app.js` and copy this code into it

```js
const http = require('http');
const os = require('os');
console.log("Kubia server starting...");
var handler = function (request, response) {
  console.log("Received request from " + request.connection.remoteAddress);
  response.writeHead(200);
  response.end("You've hit " + os.hostname() + "\n");
};
var www = http.createServer(handler);
www.listen(8080);

```

Now we will create a docker file that will run on cluster when we create a docker image. 

- create a file named `Dockerfile` and copy this code into it.

```docker
FROM node:8

RUN npm i

ADD app.js /app.js

ENTRYPOINT [ "node", "app.js" ]

```

#### Building docker image

Make sure your **docker server is up and running**. Now we will create a docker image in our local machine. Open your terminal in current project's folder and run

`docker build -t kubia .`

You’re telling Docker to build an image called **kubia** based on the contents of the current directory (note the dot at the end of the build command). Docker will look for the Dockerfile in the directory and build the image based on the instructions in the file.

Now check the your docker image created by running

#### Getting docker images

`docker images`

This command lists all the images.

#### Running the container image

`docker run --name kubia-container -p 8080:8080 -d kubia`

This tells Docker to run a new container called **kubia-container** from the kubia image. The container will be detached from the console (-d flag), which means it will run in the background. Port 8080 on the local machine will be mapped to port 8080 inside the container (-p 8080:8080 option), so you can access the app through [localhost](http://localhost:8080).

#### Accessing your application

Run in your terminal

`curl localhost:8080`
> You’ve hit 44d76963e8e1

#### Lisiting all your running containers

You can list all your running containers by this command.

`docker ps`

The `docker ps` command only shows the most basic information about the containers.

Also to get additional infomation about a container run this command

`docker inspect kubia-container`

You can see all container by

`docker ps -a`

#### Running a shell inside an existing container

The Node.js image on which you’ve based your image contains the bash shell, so you can run the shell inside the container like this:

`docker exec -it kubia-container bash`

This will run bash inside the existing **kubia-container** container. The **bash** process will have the same Linux namespaces as the main container process. This allows you to explore the container from within and see how Node.js and your app see the system when running inside the container. The **-it** option is shorthand for two options:

- -i, which makes sure STDIN is kept open. You need this for entering commands into the shell.
- -t, which allocates a pseudo terminal (TTY).

#### Exploring container from within

Let’s see how to use the shell in the following listing to see the processes running in the container.

```bash
root@c61b9b509f9a:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.4  1.3 872872 27832 ?        Ssl  06:01   0:00 node app.js
root        11  0.1  0.1  20244  3016 pts/0    Ss   06:02   0:00 bash
root        16  0.0  0.0  17504  2036 pts/0    R+   06:02   0:00 ps aux
```

You see only three processes. You don’t see any other processes from the host OS.


Like having an isolated process tree, each container also has an isolated filesystem. Listing the contents of the root directory inside the container will only show the files in the container and will include all the files that are in the image plus any files that are created while the container is running (log files and similar), as shown in the fol- lowing listing.

```bash
root@c61b9b509f9a:/# ls
app.js  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  package-lock.json  proc  root  run  sbin  srv  sys  tmp  usr  var
```

It contains the **app.js** file and other system directories that are part of the node:8 base image you’re using. To exit the container, you exit the shell by running the **exit** command and you’ll be returned to your host machine (like logging out of an ssh session, for example).

#### Stopping and removing a container

`docker stop kubia-container`

This will stop the main process running in the container and consequently stop the container, because no other processes are running inside the container. The container itself still exists and you can see it with **docker ps -a**. The -a option prints out all the containers, those running and those that have been stopped. To truly remove a container, you need to remove it with the **docker rm** command:

`docker rm kubia-container`

This deletes the container. All its contents are removed and it can’t be started again.

#### Pushing the image to an image registry

The image you’ve built has so far only been available on your local machine. To allow you to run it on any other machine, you need to push the image to an external image registry. For the sake of simplicity, you won’t set up a private image registry and will instead push the image to [Docker Hub](http://hub.docker.com)

Before you do that, you need to re-tag your image according to Docker Hub’s rules. Docker Hub will allow you to push an image if the image’s repository name starts with your Docker Hub ID. You create your Docker Hub ID by registering at [hub-docker](http://hub.docker.com). I’ll use my own ID (knrt10) in the following examples. Please change every occurrence with your own ID.

Once you know your ID, you’re ready to rename your image, currently tagged as kubia, to knrt10/kubia (replace knrt10 with your own Docker Hub ID):

`docker tag kubia knrt10/kubia`

This doesn’t rename the tag; it creates an additional tag for the same image. You can confirm this by listing the images stored on your system with the docker images command, as shown in the following listing.

`docker images | head`

As you can see, both kubia and knrt10/kubia point to the same image ID, so they’re in fact one single image with two tags.

#### Pushing image to docker hub

Before you can push the image to Docker Hub, you need to log in under your user ID with the **docker login** command. Once you’re logged in, you can finally push the yourid/kubia image to Docker Hub like this:

`docker push knrt10/kubia`

## Working with Kubernetes

Now that you have your app packaged inside a container image and made available through Docker Hub, you can deploy it in a Kubernetes cluster instead of running it in Docker directly. But first, you need to set up the cluster itself.

#### Setting up a Kubernetes cluster

Setting up a full-fledged, multi-node Kubernetes cluster isn’t a simple task, especially if you’re not well-versed in Linux and networking administration. A proper Kubernetes install spans multiple physical or virtual machines and requires the networking to be set up properly, so that all the containers running inside the Kubernetes cluster can connect to each other through the same flat networking space.

#### Running a local single-node Kubernetes cluster with Minikube

The simplest and quickest path to a fully functioning Kubernetes cluster is by using Minikube. Minikube is a tool that sets up a single-node cluster that’s great for both testing Kubernetes and developing apps locally.

#### Starting a Kubernetes cluster with minkube

Once you have Minikube installed locally, you can immediately start up the Kubernetes cluster with the command in the following listing.

`minikube start`
> Starting local Kubernetes cluster...

> Starting VM...

> SSH-ing files into VM...

> Kubectl is now configured to use the cluster.

Starting the cluster takes more than a minute, so don’t interrupt the command before
it completes.

#### Checking to see if the cluster is up and kubernetes can talk to it

To interact with Kubernetes, you also need the **kubectl** CLI client. Again, all you need to do is download it and put it on your path. 

To verify your cluster is working, you can use the **kubectl cluster-info** command shown in the following listing.

`kubectl cluster-info`
> Kubernetes master is running at https://192.168.99.100:8443

> kubernetes-dashboard is running at https://192.168.99.100:8443/api/v1/...

This shows the cluster is up. It shows the URLs of the various Kubernetes components, including the API server and the web console.

#### Deploying your Node.js app

The simplest way to deploy your app is to use the **kubectl run** command, which will create all the necessary components without having to deal with JSON or YAML.

`kubectl run kubia --image=knrt10/kubia --port=8080 --generator=run/v1`

The --image=knrt10/kubia part obviously specifies the container image you want to run, and the --port=8080 option tells Kubernetes that your app is listening on port 8080. The last flag (--generator) does require an explanation, though. Usually, you won’t use it, but you’re using it here so Kubernetes creates a **ReplicationController** instead of a Deployment.

#### Listing Pods

Because you can’t list individual containers, since they’re not standalone Kubernetes objects, can you list pods instead? Yes, you can. Let’s see how to tell kubectl to list pods in the following listing.

`kubectl get pods`

```bash
$ kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
kubia-5k788   1/1       Running   1          7d
```

#### Accessing your web application

With your pod running, how do you access it? Each pod gets its own IP address, but this address is internal to the cluster and isn’t accessible from outside of it. To make the pod accessible from the outside, you’ll expose it through a Service object. You’ll create a special service of type LoadBalancer, because if you create a regular service (a ClusterIP service), like the pod, it would also only be accessible from inside the cluster. By creating a LoadBalancer-type service, an external load balancer will be created and you can connect to the pod through the load balancer’s public IP.

#### Creating a service object

To create the service, you’ll tell Kubernetes to expose the ReplicationController you created earlier:

`kubectl expose rc kubia --type=LoadBalancer --name kubia-http`
> service "kubia-http" exposed


**Important:** We’re using the abbreviation `rc` instead of `replicationcontroller`. Most resource types have an abbreviation like this so you don’t have to type the full name (for example, `po` for `pods`, `svc` for `services`, and so on).

#### Listing Services

The expose command’s output mentions a service called `kubia-http`. Services are objects like Pods and Nodes, so you can see the newly created Service object by running the **kubectl get services | svc** command, as shown in the following listing.

`kubectl get svc`

```bash
NAME         TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1     <none>        443/TCP          7d
kubia-http   LoadBalancer   10.96.99.92   <pending>     8080:30126/TCP   7d
```

**Important** :- Minikube doesn’t support LoadBalancer services, so the service will never get an external IP. But you can access the service anyway through its external port. So external IP will always be pending in that case. When using Minikube, you can get the IP and port through which you
can access the service by running 

`minikube service kubia-http`

#### Horizontally scaling the application

You now have a running application, monitored and kept running by a Replication-Controller and exposed to the world through a service. Now let’s make additional magic happen.
One of the main benefits of using Kubernetes is the simplicity with which you can scale your deployments. Let’s see how easy it is to scale up the number of pods. You’ll increase the number of running instances to three.

Your pod is managed by a ReplicationController. Let’s see it with the kubectl get command:

`kubectl get rc`

```bash
NAME      DESIRED   CURRENT   READY     AGE
kubia     1         1         1         7d
```

#### Increasing the desired Replica count

To scale up the number of replicas of your pod, you need to change the desired replica count on the ReplicationController like this:

`kubectl scale rc kubia --replicas=3`
> replicationcontroller "kubia" scaled

You’ve now told Kubernetes to make sure three instances of your pod are always running. Notice that you didn’t instruct Kubernetes what action to take. You didn’t tell it to add two more pods. You only set the new desired number of instances and let Kubernetes determine what actions it needs to take to achieve the requested state.

#### Seeing the result of the Scale-out

Back to your replica count increase. Let’s list the ReplicationControllers again to see the updated replica count:

`kubectl get rc`

```bash
NAME      DESIRED   CURRENT   READY     AGE
kubia     3         3         3         7d
```

Because the actual number of pods has already been increased to three (as evident from the CURRENT column), listing all the pods should now show three pods instead of one:

`kubectl get pods`

```bash
NAME          READY     STATUS    RESTARTS   AGE
kubia-5k788   1/1       Running   1          7d
kubia-7zxwj   1/1       Running   1          3d
kubia-bsksp   1/1       Running   1          3d
```

As you can see, three pods exist instead of one. Currently running, but if it is pending, it would be ready in a few moments, as soon as the container image is downloaded and the container is started.

As you can see, scaling an application is incredibly simple. Once your app is running in production and a need to scale the app arises, you can add additional instances with a single command without having to install and run additional copies manually.

Keep in mind that the app itself needs to support being scaled horizontally. Kubernetes doesn’t magically make your app scalable; it only makes it trivial to scale the app up or down.

#### Displaying the Pod IP and and Pod's Node when listing Pods

If you’ve been paying close attention, you probably noticed that the **kubectl get pods** command doesn’t even show any information about the nodes the pods are scheduled to. This is because it’s usually not an important piece of information.

But you can request additional columns to display using the **-o wide** option. When listing pods, this option shows the pod’s IP and the node the pod is running on:

`kubectl get pods -o wide`

```bash
NAME          READY     STATUS    RESTARTS   AGE       IP           NODE
kubia-5k788   1/1       Running   1          7d        172.17.0.4   minikube
kubia-7zxwj   1/1       Running   1          3d        172.17.0.5   minikube
kubia-bsksp   1/1       Running   1          3d        172.17.0.6   minikube
```

#### Accessing Dashboard when using Minikube

To open the dashboard in your browser when using Minikube to run your Kubernetes cluster, run the following command:

`minikube dashboard`
