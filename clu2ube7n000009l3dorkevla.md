---
title: "Kubernetes Series Part 3 - Deploy a node application to a Kubernetes cluster"
seoTitle: "Kubernetes Series Part 3 - Deploy an app to Kubernetes"
seoDescription: "We'll deploy our node application to a Kubernetes cluster using Digital Ocean as our cloud service."
datePublished: Sat Apr 08 2023 23:00:00 GMT+0000 (Coordinated Universal Time)
cuid: clu2ube7n000009l3dorkevla
slug: kubernetes-series-part-3-deploy-a-node-application-to-a-kubernetes-cluster
canonical: https://ilearnedathing.com/post/kubernetes-series-part-3-deploy-a-node-application-to-a-kubernetes-cluster
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1711122120723/5a523ae1-41c5-484f-828d-da633c46571d.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1711122126248/716b10db-3db6-4256-9e7b-622aa27f29ba.jpeg
tags: docker, nodejs, kubernetes
domain: ilearnedathing.com

---

## 🏗️ What we’ll build

In previous posts from this series, we created an express server that serves weather data from the open weather API. We then created a Dockerfile for that service that we can use to deploy to various cloud services.

This post will deploy that Dockerfile to a Kubernetes (k8s) cluster running on Digital Ocean, a popular cloud hosting provider.

## ❓ What is Kubernetes?

Straight from the [docs](https://kubernetes.io):

> Kubernetes, also known as K8s, is an open-source system for automating the deployment, scaling, and management of containerized applications.

> It groups containers that make up an application into logical units for easy management and discovery. Kubernetes builds upon 15 years of experience of running production workloads at Google, combined with best-of-breed ideas and practices from the community.

## ✅ Prerequisites

- Read [part 1](https://www.ilearnedathing.com/post/kubernetes-series-part-1-build-an-express-api) and [part 2](https://www.ilearnedathing.com/post/kubernetes-series-part-2-containerize-a-node-application) of this series.
- A [DigitalOcean](https://digitalocean.com) account
- Basic knowledge of Docker and containers
- Install the [Kubernetes official client](https://kubernetes.io/docs/tasks/tools/) and the [Digital Ocean command-line tool](https://docs.digitalocean.com/reference/doctl/how-to/install/)

## 🌊 Set up a Kubernetes cluster on Digital Ocean

1. Log into your Digital Ocean account and choose `Kubernetes` from the side navigation.
2. Click on `Create Cluster`
3. Choose a fixed-size cluster with basic nodes. 3 nodes are fine for this exercise.
4. Click `Create Cluster` again

<aside>
⚠️ You will see that this cluster will cost around $72 per month (at the time of publishing this article). Digital Ocean charges you per minute, so the charges will be small if you delete this cluster once we’re finished with the tutorial. There is also a possibility that you were given free credits when you created your account. Free stuff is awesome!

</aside>

1. Follow the steps to authenticate with your cluster from your terminal. (Choose the automatic method)

```bash
doctl kubernetes cluster kubeconfig save <provided-cluster-key>
```

```bash
Notice: Adding cluster credentials to kubeconfig file found in "/Users/<username>/.kube/config"
Notice: Setting current-context to <cluster-name>
```

1. Test the connection to your cluster by issuing a few `kubectl` commands

```bash
kubectl get nodes
```

```bash
NAME                     STATUS   ROLES    AGE   VERSION
node-weather-app-q1u1d   Ready    <none>   27m   v1.26.3
node-weather-app-q1u1i   Ready    <none>   27m   v1.26.3
node-weather-app-q1u1v   Ready    <none>   27m   v1.26.3
```

1. Click `I'm done`. You now have a Kubernetes cluster running. Now we’re on our own 😄

## 🗒️ Create a Kubernetes deployment for your Express application.

In order to deploy our Express server to our Kubernetes cluster, we need to create a Deployment and a Service.

Create a new file called `deployment.yml` with the following contents:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: express-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: express-server
  template:
    metadata:
      labels:
        app: express-server
    spec:
      containers:
      - name: express-server
        image: your-dockerhub-username/<express-server-image-name>:latest
        ports:
        - containerPort: 3000
        env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: api-key-secret
              key: api_key

---

apiVersion: v1
kind: Service
metadata:
  name: express-server
spec:
  selector:
    app: express-server
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

The `deployment.yml` file contains two Kubernetes resources:

1. Deployment: This specifies the desired state for the Express server application, such as the number of replicas and the container image to use.
2. Service: This resource creates a load balancer that listens on port 80 and forwards traffic to the application on port 3000.

<aside>
💡 `API_KEY`: This value is fetched from a Kubernetes secret named `api-key-secret`. Using secrets for sensitive data is a recommended best practice in Kubernetes.

</aside>

To create the `api-key-secret`, run the following command:

```bash
kubectl create secret generic api-key-secret --from-literal=api_key=your-api-key-value
```

## 🚀 Deploy to your Kubernetes cluster

With the `deployment.yml` file in place, you can deploy your Express server to your Kubernetes cluster. First, ensure you have the correct context set with `kubectl` by running:

```bash
kubectl config use-context <your-kubernetes-context>
```

Then, deploy your application by running:

```bash
kubectl apply -f deployment.yml
```

## 🌍 Access your application on the internet

To make your Express server accessible on the internet, the `Service` resource you created earlier has exposed it via a load balancer. To get the external IP address of the load balancer, run the following command:

```bash
kubectl get service express-server
```

The output should look similar to the following:

```bash
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
express-server  LoadBalancer   10.47.248.217   203.0.113.42      80:32426/TCP   5m
```

In this example, the external IP address is `203.0.113.42`. You can now access your Express server on the internet by navigating to `http://203.0.113.42`.

## 💡 Troubleshooting

If you built your images on an M1 Macbook, or a machine with ARM architecture, you may run into the issue of your pods crashing when you deploy them.

To avoid this issue, you will need to build your images on your MacBook with `buildx`.

Run the following commands to build you images with `buildx`:

```bash
docker buildx install

docker buildx create --name multiarch-builder

docker buildx use multiarch-builder

docker buildx build --platform linux/amd64,linux/arm64 --tag your-dockerhub-username/<express-server-image-name>:latest --push .
```

This will produce a multi-architecture build of your image that *should* now work on Linux Digital Ocean droplets.

## That’s it!

In this blog post, we have walked through the process of deploying an Express.js server to a Kubernetes cluster and making it available on the internet. This process involved containerizing the application using Docker, creating Kubernetes deployment and service resources, deploying the application to a Kubernetes cluster, and accessing it via an external IP address.

By following these steps, you can leverage the power of Kubernetes to manage and scale your Express.js applications easily and efficiently.

In the next post, we’ll learn how to use a vault to have more control over our secrets.
