---
title: Deploying a snake game using Kubernetes
date: 2023-06-11 17:45:07 +07:00
tags: [aws]
description: A blog about deploying a snake game using Kubernetes
---

A long time back, I had deployed a regular Snake game in a Docker container. Creating an image, deployment of a container and then playing the game (of course) was a good learning experience. This time around, I decided to take it on step further and deploy the same game with Kubernetes. Learning Kubernetes is daunting but as they say eat an elephant one bite at a time. Here are the main components required for the game to work in Kubernetes PODs.

- Deployment - A deployment mainly contains any node affinity rules, the number of desired replicas, and the image details from where the PODs will be built. Here is a sample deployment file for reference:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: snake-game-deployment
  namespace: kube-system
  labels:
    app: snake-game
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: snake-game
  template:
    metadata:
      labels:
        app.kubernetes.io/name: snake-game
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:
                - amd64
                - arm64
      containers:
      - name: snake-game
        image: rahulgadre/snake-game
        ports:
        - name: http
          containerPort: 80
        imagePullPolicy: IfNotPresent
```

- Service - A service is like a load balancer which sends traffic to the backend PODs. Things are tied together in Kubernetes mainly with tags. Hence, ensure that the tags used in the deployment, service, and ingress match. Refer to the official Kubernetes documentation for further information Node types.

```
apiVersion: v1
kind: Service
metadata:
  name: snake-service
  namespace: kube-system
  labels:
    app: snake-game
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
  selector:
    app.kubernetes.io/name: snake-game
```
- Ingress - The last required piece for the game to work is Ingress which is an external facing load balancer that accepts traffic from the outside of the Kubernetes cluster.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: kube-system
  name: snake-game-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: snake-service
              port:
                number: 80
```
Once the deployment is created and the service is deployed, an ingress needs to be deployed. In my case, it was deployed with the command "kubectl delete ingress snake-game-ingress -n kube-system". Once the ingress was deployed, it created a load balancer in AWS and the DNS name of the ALB was then used to play the game.

As described in the deployment, kubernetes deployed 2 snake-game PODs.
```
kube-system    snake-game-deployment-575dff499d-ckm6p          1/1     Running   2 (14d ago)    111d   192.168.41.179   ip-192-168-49-254.us-west-2.compute.internal   <none>           <none>
kube-system    snake-game-deployment-575dff499d-rwd27          1/1     Running   2 (14d ago)    111d   192.168.89.237   ip-192-168-65-188.us-west-2.compute.internal   <none>           <none>
```
The screenshot below shows the working game in a browser which is running on Kubernetes PODs which are behing an Ingress.

 ![](/assets/img/snake-game.png)
