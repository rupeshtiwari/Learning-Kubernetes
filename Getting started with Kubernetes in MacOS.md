# Getting started with Kubernetes on your local macOS. 
🌟 Are you ready to dive into the world of Kubernetes from scratch? 💻

🚀 Calling all aspiring developers! Whether you're an Angular, .NET, or Java enthusiast, a frontend or backend guru, or a full stack enthusiast, this step-by-step beginner's guide is perfect for you! 🎉

📚 In this comprehensive tutorial, we will start from the very basics and guide you through the process of getting started with Kubernetes. You don't need any prior knowledge of Kubernetes or containerization - we'll cover everything from the ground up. You'll learn about the core concepts, containerization, Kubernetes architecture, and how to deploy and manage your applications using Kubernetes.

So, if you're excited to learn Kubernetes and take your application development skills to the next level, let's dive in and embark on this Kubernetes journey together! 🌈✨

## Installing dependencies: Docker Desktop for MacOS



Before we proceed with setting up Minikube, it is essential to ensure that Docker is installed and running on your macOS machine. Here are the steps to [install Docker Desktop on macOS](https://gist.github.com/rupeshtiwari/5be977fab589ba3de279177000f9160a) and start the Docker service:


<img width="994" alt="image" src="https://user-images.githubusercontent.com/330383/252155807-9ede8eae-86b8-4079-a8c9-a3b910a21cde.png">

Next run docker to see if your docker is running

<img width="764" alt="image" src="https://user-images.githubusercontent.com/330383/252155839-b875b425-4405-43b4-987f-9d10de58490f.png">


## Installing minikube
[Minikube](https://minikube.sigs.k8s.io/docs/start/) is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.

```
curl -LO https://github.com/kubernetes/minikube/releases/download/v1.30.1/minikube-darwin-arm64  
sudo install minikube-darwin-arm64 /usr/local/bin/minikube

```

Start Minikube, before starting minikube make sure docker is running by openeing docker desktop on mac. Minikube start may take few minutes to generate the cluster. 


```
minikube start
```

<img width="1336" alt="image" src="https://user-images.githubusercontent.com/330383/252155658-6f6deb74-076a-460b-a55b-4ce97049213b.png">

Next lets explore you Kubernetes(K8) **cluster** 

```
kubectl cluster-info
```
You will see the ip address as localhost address since this cluster is setup on your machine running at ` https://127.0.0.1:53273`

<img width="1307" alt="image" src="https://user-images.githubusercontent.com/330383/252155711-725de4c5-ae3f-4c86-a333-c29f01d8b40a.png">

Next check all the **nodes**

```
kubectl get nodes
```
<img width="1205" alt="image" src="https://user-images.githubusercontent.com/330383/252155990-85d9445b-02ae-426d-bbe4-4dc5c82f16ab.png">

We get the error that means our cluster is not running yet. Let's run `minikube start` one more time. And then re-run `kubectl get nodes`

<img width="1233" alt="image" src="https://user-images.githubusercontent.com/330383/252156079-032b29f4-5039-4c65-85e7-271f8a7a9460.png">

One node with name `minikube` with role of `control-plane` my k8 is running with version `1.26`. 

Let's check **namespaces** created by default 

```
kubectl get namespaces
```
<img width="578" alt="image" src="https://user-images.githubusercontent.com/330383/252156199-a5774e3c-de58-41cc-a209-d7f2b791f1cc.png">

You isolate and manage applications and services using namespace. 

Next let's see the **pods** created in all namespaces by passing `-A`. 

```
kubectl get pods -A
```

<img width="1111" alt="image" src="https://user-images.githubusercontent.com/330383/252156327-278e1f93-f9f3-4145-af1c-69f87562fb9e.png">

These pods, how containers are running k8 these pods have softwares to run k8 cluster itself. 

Now let's see all the **services**. Services act as the loadbalancer and direct the traffic to the pods. 

```
kubectl get services -A
```
<img width="1155" alt="image" src="https://user-images.githubusercontent.com/330383/252156479-86238b5d-49e4-4c08-807e-83a0825ad751.png">

## Creating Namespaces 

If you are using single cluster for your applicaiton then You can `namespace` to isolate and organize workloads. Example create `dev` and `prod` namespaces to separate them.

Check existing namespaces 
```
kubectl get namespaces 
```

K8 menefest to create a namespace called `development` in file `namespace.yml`

```yml
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

Let's create the namesapce by runing command 

```
kubectl apply -f namespace.yaml
```

<img width="901" alt="image" src="https://user-images.githubusercontent.com/330383/252158462-81b40cfd-0721-4df4-a193-f61fd33ce111.png">

Let's change the namespace.yaml to have both production and development namespaces and run  `
kubectl apply -f namespace.yaml` to create both of them. 

```yml
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
--- # way to signify new document in yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

<img width="932" alt="image" src="https://user-images.githubusercontent.com/330383/252158538-b719f90c-b007-4e92-8e41-59f65dc2d84b.png">

Now let's delete these 2 namespaces 

```
kubectl delete -f namespace.yaml
```
<img width="941" alt="image" src="https://user-images.githubusercontent.com/330383/252158640-cf4dcdcc-dd2b-4625-acca-245f3407c9db.png">

## Deploy an Application 

Let's create Highly available application that will be deployed in 3 different pods. We will declare our dpplicaiton inside the `deployment.yaml` file how we want our application to be deployed in `development` namespace, with `3` replicas. We will run container in the pod which will display the information of the pod that it is running within you will use `kimschles/pod-info-app:latest` image that will have that pod info app already created. We will setup container with 3 environment variables for pod name, namespace and ip that will be used by our app. 

Let's create the `development` namespace first. Run previous command `kubectl apply -f namespace.yaml` to deploy namespace. 

<img width="923" alt="image" src="https://user-images.githubusercontent.com/330383/252159179-a003ade5-aa41-47e8-a409-53efca8a6f09.png">

Next create `deployment.yaml` with below definition. 

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-info-deployment
  namespace: development
  labels:
    app: pod-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-info
  template:
    metadata:
      labels:
        app: pod-info
    spec:
      containers:
      - name: pod-info-container
        image: kimschles/pod-info-app:latest
        ports:
        - containerPort: 3000
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace                
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP        
 ```
We do not have any deployment in our cluster yet `kubectl get deployments`

<img width="1085" alt="image" src="https://user-images.githubusercontent.com/330383/252159256-fc95561c-d3f4-4070-baa7-04e862357e38.png">

Now Let's create the deployment 

```
kubectl apply -f deployment.yaml
```

Next let's check if the deployment is created `kubectl get deployments -n development` 

<img width="1009" alt="image" src="https://user-images.githubusercontent.com/330383/252159395-87358de1-d78c-45c3-9645-309b8576d791.png">


⚠️ Notice `pod-info-deployment` is created with 3 containers not ready yet after `15s` but wait for few more time and again check you should have 3 pods ready to go and available 

<img width="1068" alt="image" src="https://user-images.githubusercontent.com/330383/252159558-be2c6776-d1d9-4c75-ae6d-844c546129d0.png">

Next let's check all the pods in the `development` namespace

```
kubectl get pods -n development 
```
<img width="954" alt="image" src="https://user-images.githubusercontent.com/330383/252159622-d65d822f-3738-4826-adf6-acb3e475d273.png">

Notice all the pods are running. 

You have set the replica as `3` so always 3 pods will be running. To proove that go ahead and delete a pod and check. 

Delete first pod by copy it's name and use `delete pod` command. 

```
kubectl delete pod pod-info-deployment-757cb75bbb-l2jd8 -n development 
```

Check how much pods you left `kubectl get pods -n development`

<img width="1394" alt="image" src="https://user-images.githubusercontent.com/330383/252159877-18c23179-d84c-4c3f-88e3-fbf10f3ad931.png">

😲 You see still 3 pods, K8 deployment automatically terminated the one that you asked to delete and created brand new pod to make sure at any time 3 pods are running. 

🎉 Pet yourself 🐶, you created 3 pods with k8 deployment. 

## How to check health of your POD? 

You check the health of a pod by looking at the event logs. Why pods may not work: container image not available, Worker nodes are out of space so pod can not be schedule. 

```
# Find the pod that you want to check logs 
kubectl get pods -n development
 
kubectl describe pod <pod-name> -n development

# We will take first pod and check its logs 
kubectl describe pod pod-info-deployment-757cb75bbb-8cgf8 -n development

```
 <img width="1440" alt="image" src="https://user-images.githubusercontent.com/330383/252160272-0f65eb3b-f3c4-4c03-86a4-6ef42a8f520d.png">


If your pod is not working check at the bottom and you will see the events with error messages. 

## How to check your application is Working?

You can use [BusyBox](https://busybox.net/) tool to debug and troubleshoot issues in linux environment. For this you have to re-deploy your application with busybox tool. 

In below definition `busybox.yaml` of deployment, we will use 1 replica and deploy application in `default` namespace. We will pull the image from `busybox:latest`.  

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
  namespace: default
  labels:
    app: busybox
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox-container
        image: busybox:latest
        # Keep the container running
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 30; done;" ]
        resources:
          requests:
            cpu: 30m
            memory: 64Mi
          limits:
            cpu: 100m
            memory: 128Mi
 ```
Deploy it 

```
kubectl apply -f busybox.yaml
```
<img width="875" alt="image" src="https://user-images.githubusercontent.com/330383/252178510-8253388b-8181-4524-aa13-3d8058334f7a.png">

Get pods `kubectl get pods`

# Gettings started with Kubernetes on your local macOS. 
🌟 Are you ready to dive into the world of Kubernetes from scratch? 💻

🚀 Calling all aspiring developers! Whether you're an Angular, .NET, or Java enthusiast, a frontend or backend guru, or a full stack enthusiast, this step-by-step beginner's guide is perfect for you! 🎉

📚 In this comprehensive tutorial, we will start from the very basics and guide you through the process of getting started with Kubernetes. You don't need any prior knowledge of Kubernetes or containerization - we'll cover everything from the ground up. You'll learn about the core concepts, containerization, Kubernetes architecture, and how to deploy and manage your applications using Kubernetes.

So, if you're excited to learn Kubernetes and take your application development skills to the next level, let's dive in and embark on this Kubernetes journey together! 🌈✨

## Installing dependencies: Docker Desktop for MacOS



Before we proceed with setting up Minikube, it is essential to ensure that Docker is installed and running on your macOS machine. Here are the steps to [install Docker Desktop on macOS](https://gist.github.com/rupeshtiwari/5be977fab589ba3de279177000f9160a) and start the Docker service:


<img width="994" alt="image" src="https://user-images.githubusercontent.com/330383/252155807-9ede8eae-86b8-4079-a8c9-a3b910a21cde.png">

Next run docker to see if your docker is running

<img width="764" alt="image" src="https://user-images.githubusercontent.com/330383/252155839-b875b425-4405-43b4-987f-9d10de58490f.png">


## Installing minikube
[Minikube](https://minikube.sigs.k8s.io/docs/start/) is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.

```
curl -LO https://github.com/kubernetes/minikube/releases/download/v1.30.1/minikube-darwin-arm64  
sudo install minikube-darwin-arm64 /usr/local/bin/minikube

```

Start Minikube, before starting minikube make sure docker is running by openeing docker desktop on mac. Minikube start may take few minutes to generate the cluster. 


```
minikube start
```

<img width="1336" alt="image" src="https://user-images.githubusercontent.com/330383/252155658-6f6deb74-076a-460b-a55b-4ce97049213b.png">

Next lets explore you Kubernetes(K8) **cluster** 

```
kubectl cluster-info
```
You will see the ip address as localhost address since this cluster is setup on your machine running at ` https://127.0.0.1:53273`

<img width="1307" alt="image" src="https://user-images.githubusercontent.com/330383/252155711-725de4c5-ae3f-4c86-a333-c29f01d8b40a.png">

Next check all the **nodes**

```
kubectl get nodes
```
<img width="1205" alt="image" src="https://user-images.githubusercontent.com/330383/252155990-85d9445b-02ae-426d-bbe4-4dc5c82f16ab.png">

We get the error that means our cluster is not running yet. Let's run `minikube start` one more time. And then re-run `kubectl get nodes`

<img width="1233" alt="image" src="https://user-images.githubusercontent.com/330383/252156079-032b29f4-5039-4c65-85e7-271f8a7a9460.png">

One node with name `minikube` with role of `control-plane` my k8 is running with version `1.26`. 

Let's check **namespaces** created by default 

```
kubectl get namespaces
```
<img width="578" alt="image" src="https://user-images.githubusercontent.com/330383/252156199-a5774e3c-de58-41cc-a209-d7f2b791f1cc.png">

You isolate and manage applications and services using namespace. 

Next let's see the **pods** created in all namespaces by passing `-A`. 

```
kubectl get pods -A
```

<img width="1111" alt="image" src="https://user-images.githubusercontent.com/330383/252156327-278e1f93-f9f3-4145-af1c-69f87562fb9e.png">

These pods, how containers are running k8 these pods have softwares to run k8 cluster itself. 

Now let's see all the **services**. Services act as the loadbalancer and direct the traffic to the pods. 

```
kubectl get services -A
```
<img width="1155" alt="image" src="https://user-images.githubusercontent.com/330383/252156479-86238b5d-49e4-4c08-807e-83a0825ad751.png">

## Creating Namespaces 

If you are using single cluster for your applicaiton then You can `namespace` to isolate and organize workloads. Example create `dev` and `prod` namespaces to separate them.

Check existing namespaces 
```
kubectl get namespaces 
```

K8 menefest to create a namespace called `development` in file `namespace.yml`

```yml
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

Let's create the namesapce by runing command 

```
kubectl apply -f namespace.yaml
```

<img width="901" alt="image" src="https://user-images.githubusercontent.com/330383/252158462-81b40cfd-0721-4df4-a193-f61fd33ce111.png">

Let's change the namespace.yaml to have both production and development namespaces and run  `
kubectl apply -f namespace.yaml` to create both of them. 

```yml
---
apiVersion: v1
kind: Namespace
metadata:
  name: development
--- # way to signify new document in yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

<img width="932" alt="image" src="https://user-images.githubusercontent.com/330383/252158538-b719f90c-b007-4e92-8e41-59f65dc2d84b.png">

Now let's delete these 2 namespaces 

```
kubectl delete -f namespace.yaml
```
<img width="941" alt="image" src="https://user-images.githubusercontent.com/330383/252158640-cf4dcdcc-dd2b-4625-acca-245f3407c9db.png">

## Deploy an Application 

Let's create Highly available application that will be deployed in 3 different pods. We will declare our dpplicaiton inside the `deployment.yaml` file how we want our application to be deployed in `development` namespace, with `3` replicas. We will run container in the pod which will display the information of the pod that it is running within you will use `kimschles/pod-info-app:latest` image that will have that pod info app already created. We will setup container with 3 environment variables for pod name, namespace and ip that will be used by our app. 

Let's create the `development` namespace first. Run previous command `kubectl apply -f namespace.yaml` to deploy namespace. 

<img width="923" alt="image" src="https://user-images.githubusercontent.com/330383/252159179-a003ade5-aa41-47e8-a409-53efca8a6f09.png">

Next create `deployment.yaml` with below definition. 

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-info-deployment
  namespace: development
  labels:
    app: pod-info
spec:
  replicas: 3
  selector:
    matchLabels:
      app: pod-info
  template:
    metadata:
      labels:
        app: pod-info
    spec:
      containers:
      - name: pod-info-container
        image: kimschles/pod-info-app:latest
        ports:
        - containerPort: 3000
        env:
          - name: POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace                
          - name: POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP        
 ```
We do not have any deployment in our cluster yet `kubectl get deployments`

<img width="1085" alt="image" src="https://user-images.githubusercontent.com/330383/252159256-fc95561c-d3f4-4070-baa7-04e862357e38.png">

Now Let's create the deployment 

```
kubectl apply -f deployment.yaml
```

Next let's check if the deployment is created `kubectl get deployments -n development` 

<img width="1009" alt="image" src="https://user-images.githubusercontent.com/330383/252159395-87358de1-d78c-45c3-9645-309b8576d791.png">


⚠️ Notice `pod-info-deployment` is created with 3 containers not ready yet after `15s` but wait for few more time and again check you should have 3 pods ready to go and available 

<img width="1068" alt="image" src="https://user-images.githubusercontent.com/330383/252159558-be2c6776-d1d9-4c75-ae6d-844c546129d0.png">

Next let's check all the pods in the `development` namespace

```
kubectl get pods -n development 
```
<img width="954" alt="image" src="https://user-images.githubusercontent.com/330383/252159622-d65d822f-3738-4826-adf6-acb3e475d273.png">

Notice all the pods are running. 

You have set the replica as `3` so always 3 pods will be running. To proove that go ahead and delete a pod and check. 

Delete first pod by copy it's name and use `delete pod` command. 

```
kubectl delete pod pod-info-deployment-757cb75bbb-l2jd8 -n development 
```

Check how much pods you left `kubectl get pods -n development`

<img width="1394" alt="image" src="https://user-images.githubusercontent.com/330383/252159877-18c23179-d84c-4c3f-88e3-fbf10f3ad931.png">

😲 You see still 3 pods, K8 deployment automatically terminated the one that you asked to delete and created brand new pod to make sure at any time 3 pods are running. 

🎉 Pet yourself 🐶, you created 3 pods with k8 deployment. 

## How to check health of your POD? 

You check the health of a pod by looking at the event logs. Why pods may not work: container image not available, Worker nodes are out of space so pod can not be schedule. 

```
# Find the pod that you want to check logs 
kubectl get pods -n development
 
kubectl describe pod <pod-name> -n development

# We will take first pod and check its logs 
kubectl describe pod pod-info-deployment-757cb75bbb-8cgf8 -n development

```
 <img width="1440" alt="image" src="https://user-images.githubusercontent.com/330383/252160272-0f65eb3b-f3c4-4c03-86a4-6ef42a8f520d.png">


If your pod is not working check at the bottom and you will see the events with error messages. 

## How to check your application is Working?

You can use [BusyBox](https://busybox.net/) tool to debug and troubleshoot issues in linux environment. For this you have to re-deploy your application with busybox tool. 

In below definition `busybox.yaml` of deployment, we will use 1 replica and deploy application in `default` namespace. We will pull the image from `busybox:latest`.  

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
  namespace: default
  labels:
    app: busybox
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox-container
        image: busybox:latest
        # Keep the container running
        command: [ "/bin/sh", "-c", "--" ]
        args: [ "while true; do sleep 30; done;" ]
        resources:
          requests:
            cpu: 30m
            memory: 64Mi
          limits:
            cpu: 100m
            memory: 128Mi
 ```
Deploy it 

```
kubectl apply -f busybox.yaml
```
<img width="875" alt="image" src="https://user-images.githubusercontent.com/330383/252178510-8253388b-8181-4524-aa13-3d8058334f7a.png">

Get pods `kubectl get pods`

<img width="738" alt="image" src="https://user-images.githubusercontent.com/330383/252178837-78ffc900-d8f1-4044-bf3d-24d5820001c7.png">

Open new terminal and fetch all pods in development with its ip details `-o wide` gives pods extra info like IP address 
```
kubectl get pods -n development -o wide 
```
<img width="1389" alt="image" src="https://user-images.githubusercontent.com/330383/252179143-09a467b9-327f-4694-b70c-1eb1906183e2.png">

Next, run command to get into the busybox pod and open bash shell. Copy the busybox pod name from previous open terminal.  

```
kubectl exec -it busybox-6b95744666-zh9tx -- /bin/sh
```

You will see command shell type `wget` to see it is installed or not. 

<img width="1178" alt="image" src="https://user-images.githubusercontent.com/330383/252179327-e6e11393-a503-4b2e-a15e-e1bd46f46c4c.png">

Next from busybox pod I will try to `wget` one of the IP address from development pod. Copy one of the IP of the pod from development namespace and run below script on busybox interactive terminal. 

```
wget 10.244.0.13
```
You will get connection refused error

<img width="884" alt="image" src="https://user-images.githubusercontent.com/330383/252179746-5b7de116-5dbb-4989-80ce-74d187fd1dff.png">

Notice that busybox is trying to access pod with default port 80 `(10.244.0.13:80)` however, our pod in development is deployed in port 3000 so lets change the script to include port number 3000 and run in busybox terminal
```
wget 10.244.0.13:3000
```
You are able to connect good news 
<img width="1427" alt="image" src="https://user-images.githubusercontent.com/330383/252180190-e1d366cd-5867-4012-a4e9-99d972629294.png">

Check the output of the pod application saved in the `index.html` file. Wget saved info in the html file. 

```
cat index.html
```
<img width="1355" alt="image" src="https://user-images.githubusercontent.com/330383/252180261-561fbfac-4de7-4297-81ad-f15205ac982a.png">


So you deployed an application in development namespace and used busybox to check if it is working or not.  Next you will learn how to check application logs. 

Run `exit` to go out of terminal  of pod. 

## View Application logs 
If you want to debug application issues use application logs. 

```
kubectl get pods -n development
```

Inspect log

```
kubectl logs <pod-name> -n development
kubectl logs pod-info-deployment-757cb75bbb-pdtxc -n development
```
<img width="1320" alt="image" src="https://user-images.githubusercontent.com/330383/252180591-a930d067-4ff5-45c2-ae01-cd6c5ab97f2e.png">

So we were able to access our pods using busybox however how do you access your pods from internet? You will learn next. 

## Expose your application to the internet with a LoadBalancer
K8 Service is a LoadBalancer service, it directs traffic from internet to pods. Loadbalancer service has public and private static IP addess. We had deployed pod with selector `pod-info`. 

There are 3 types of kubernetes services: 1) LoadBalancer, 2) Nodeport 3) ClusterIP. 

You will create LoadBalancer type service by creating `service.yaml` and running it. 

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: demo-service
  namespace: development
spec:
  selector:
    app: pod-info
  ports:
    - port: 80
      targetPort: 3000
  type: LoadBalancer
    
```
Creae Service

```
kubectl apply -f service.yaml
```
Get Service external/internal IP 
```
kubectl get services -n development
```
<img width="993" alt="image" src="https://user-images.githubusercontent.com/330383/252181670-de91c8f9-089f-4e17-8cc1-6b3c38a810d7.png">

Next in order to get external ip lets run below command in another terminal 

```
sudo minikube tunnel 
```
Next rerun service command 
```
kubectl get services -n development
```
Notice external IP is displayed as localhost. Since minikube has setup your cluster in localbox it will use localhost as external IP. If you create Loadbalancer from GCP, Azure or AWS then you would get real external internet exposed IP address. 

<img width="976" alt="image" src="https://user-images.githubusercontent.com/330383/252181761-b73b6036-3f55-49bf-9cf1-cfb93aeb0de0.png">

Next, you will access the IP address to visit http://127.0.0.1 from your laptop. Notice site not loading so go to the terminal where you run tunnel command there you have to enter your password to start this app and refresh the browser again. 

<img width="1377" alt="image" src="https://user-images.githubusercontent.com/330383/252182308-60b4ac41-e122-4c32-8679-f6b505833cb6.png">


http://127.0.0.1  site is giving the output where it shows the pod details. 
<img width="605" alt="image" src="https://user-images.githubusercontent.com/330383/252182405-714cd6dc-ede6-4b7a-a734-6176a40ad4c8.png">

You have used k8 loadbalancer service to expose the pod into the internet. Next you will restrict the resource request limit on the container in the pods. 




