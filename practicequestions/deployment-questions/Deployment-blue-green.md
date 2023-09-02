# Deployment strategies questions (use [killerkoda.com](https://killercoda.com/playgrounds/scenario/kubernetes) for practice)

## Please user alias k=kubectl to assign kubectl alias to k, You can use kubectl as well

Q. Given deployment **practicedep** with 10 replicas in **default** namespace, running with **nginx:1.14.1**. Current deployment has the label version=v1. Task is to update image to **nginx:1.14.2** using blue-green
strategy. New deployment should have label version=v2. Service is exposed for testing(attached yaml). When deployment is completed, service should start sending traffic to new deplyment.

deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: bgdeployment
      name: bgdeployment
      namespace: default
    spec:
      replicas: 10
      selector:
        matchLabels:
          app: practicedep
          version: v1
      strategy:
        rollingUpdate:
          maxSurge: 25%
          maxUnavailable: 25%
        type: RollingUpdate
      template:
        metadata:
          labels:
            app: practicedep
            version: v1
        spec:
          containers:
          - image: nginx:1.14.1
            imagePullPolicy: IfNotPresent
            name: nginx
          restartPolicy: Always

service.yaml

    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: bgservice 
      name: bgservice
      namespace: default
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        app: practicedep
        version: v1
      type: ClusterIP

You can create above resources using 

    k create -f https://raw.githubusercontent.com/letonkargit/CKAD-practice/main/resources/deployment-bg.yaml

  <Details><summary>Answer</summary>
  
  Firstly(optional step anyways), let us check if given deployment and service work correctly -

      k run temppod --image=nginx:alpine --restart=Never --rm -i -- curl http://bgservice:80

  This should return something like this(that is, it works)

      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
      <!DOCTYPE html>
      <html>
      <head>
      <title>Welcome to nginx!</title>
      <style>
          body {
              width: 35em;
              margin: 0 auto;
              font-family: Tahoma, Verdana, Arial, sans-serif;
          }
      </style>
      </head>
      <body>
      <h1>Welcome to nginx!</h1>
      <p>If you see this page, the nginx web server is successfully installed and
      working. Further configuration is required.</p>
      
      <p>For online documentation and support please refer to
      <a href="http://nginx.org/">nginx.org</a>.<br/>
      Commercial support is available at
      <a href="http://nginx.com/">nginx.com</a>.</p>
      
      <p><em>Thank you for using nginx.</em></p>
      </body>
      </html>
      100   612  100   612    0     0   186k      0 --:--:-- --:--:-- --:--:--  199k
      pod "temppod" deleted

  The step to release/upgrade app using blue-green deployment is to have similar deployment to live one with upgraded/required changes. Once copy of live 
  deployment is up and ready, we switch the traffic of service from live deployment to new deployment, since services work based on label matching, 
  we can simply update the service matchLabels property to reroute traffic to new deployment.
  We are going to follow exact similar steps - Create similar deployment with updated image(with label version=v2), test it, if everything looks good, 
  update service to start routing traffic to version=v2.

  Bring up new deployment with updated image and label(copy and edit given deployment) -

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: bgdeployment
      name: newbgdeployment
      namespace: default
    spec:
      replicas: 10
      selector:
        matchLabels:
          app: practicedep
          version: v2
      strategy:
        rollingUpdate:
          maxSurge: 25%
          maxUnavailable: 25%
        type: RollingUpdate
      template:
        metadata:
          labels:
            app: practicedep
            version: v2
        spec:
          containers:
          - image: nginx:1.14.2
            imagePullPolicy: IfNotPresent
            name: nginx
          restartPolicy: Always

  You can run below command to run new deployment -

      k create -f https://raw.githubusercontent.com/letonkargit/CKAD-practice/main/resources/deployment-bg-v2.yaml

  Run below commands to make sure deployment is up -

      k get deployments
      
      NAME              READY   UP-TO-DATE   AVAILABLE   AGE
      bgdeployment      10/10   10           10          6m12s
      newbgdeployment   10/10   10           10          2m12s

  Create service to test new deployment

    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: bgservice 
      name: bgservice
      namespace: default
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        app: practicedep
        version: v1
      type: ClusterIP

  You can run below command to create the service to test new deployment 

      k create -f https://raw.githubusercontent.com/letonkargit/CKAD-practice/main/resources/service-bg-v2.yaml

  Run below command to test new deployment

      k run temppod --image=nginx:alpine --restart=Never --rm -i -- curl http://newbgservice:80

  This should return something like this(that is, it works)
  
      % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                       Dload  Upload   Total   Spent    Left  Speed
      100   612  100   612    0     0   282k      0 --:--:-- --:--:-- --:--:--  298k
      <!DOCTYPE html>
      <html>
      <head>
      <title>Welcome to nginx!</title>
      <style>
          body {
              width: 35em;
              margin: 0 auto;
              font-family: Tahoma, Verdana, Arial, sans-serif;
          }
      </style>
      </head>
      <body>
      <h1>Welcome to nginx!</h1>
      <p>If you see this page, the nginx web server is successfully installed and
      working. Further configuration is required.</p>
      
      <p>For online documentation and support please refer to
      <a href="http://nginx.org/">nginx.org</a>.<br/>
      Commercial support is available at
      <a href="http://nginx.com/">nginx.com</a>.</p>
      
      <p><em>Thank you for using nginx.</em></p>
      </body>
      </html>
      pod "temppod" deleted

Once tested, delete newly created service `k delete svc newbgservice`

Since we made sure that new deployment runs fine, we will update existing service to start serving with new deployment.
You can run below command and edit `selector`

      k edit service bgservice

Update(version: v2)
Before:
  
  ![image](https://github.com/letonkargit/CKAD-practice/assets/32503981/6acec9b4-cd57-4721-8dc0-5cc135827b97)

After:

  ![image](https://github.com/letonkargit/CKAD-practice/assets/32503981/8501ca4f-bd1c-48a6-b974-fa2373447454)

Now, test using `k run temppod --image=nginx:alpine --restart=Never --rm -i -- curl http://newbgservice:80` 
and delete the old deployment -

    k delete deployment bgdeployment
