# bluegreen-demo-app
Blue Green Deployment with Kubernetes

We will be using **minikube** to run kubernetes locally with **Virtual Box** .

```$ minikube start --vm-driver=virtualbox```

## Procedures
1. Blue Deployemnt
1. Green Deployment
1. Exposing with a Service
1. Making Changes

### 1. Blue Deployment

Let's start by creating the demo app for *BLUE* version.
Docker image credits to **smesch/hugo-app**

```$ kubectl apply -f blue.yaml```

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webserver-blue
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: bluegreen-demo-app
        type: webserver
        color: blue
    spec:
      containers:
      - image: smesch/hugo-app:blue
        name: webserver-container
        ports:
        - containerPort: 80
          name: http-server
```

### 2. Green Deployment

Next, we will make our *GREEN* app.

```$ kubectl apply -f green.yaml```

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: webserver-green
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: bluegreen-demo-app
        type: webserver
        color: green
    spec:
      containers:
      - image: smesch/hugo-app:green
        name: webserver-container
        ports:
        - containerPort: 80
          name: http-server
```

### 3. Expose with a service

To expose the app, we will create a kubernetes Service.
Please take note the Service type **NodePort** to expose our service from internal cluster(virtual box) to host machine(local). 

```$ kubectl apply -f service.yaml```

```yaml
apiVersion: v1
kind: Service
metadata: 
  name: bluegreen-demo-app
  labels: 
    name: bluegreen-demo-app
spec:
  ports:
    - name: http
      port: 80
      targetPort: 80
  selector: 
    app: bluegreen-demo-app
    color: blue
  type: NodePort
```

Now, we can see our running service.

```$ minikube service bluegreen-demo-app```

Currently, Service is routing traffic to *BLUE* version of our application. (You can see app page with Blue header)

### 4. Making Changes

We can do *Rolling Updates* to our Service by switching to *GREEN* version.

```$ kubectl patch service bluegreen-demo-app -p '{"spec":{"selector":{"color":"green"}}}' ```

Reloading the browser will show the *GREEN* version (with Green header)
