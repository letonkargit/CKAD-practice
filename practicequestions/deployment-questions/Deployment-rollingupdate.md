# Deployment strategies questions (use [killerkoda.com](https://killercoda.com/playgrounds/scenario/kubernetes) for practice)

## Please user alias k=kubectl to assign kubectl alias to k, You can use kubectl as well

Q. Given deployment **practicedep** with 10 replicas in **default** namespace, running with **nginx:1.14.1**. Task is to update image to **nginx:1.14.2** using rolling update strategy.
   Maximum pods those can go down are 2 and maximum pods can surge are 2. Observe the updates. Service is exposed for testing(attached yaml)

deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: rollingdeployment
      name: rollingdeployment
      namespace: default
    spec:
      replicas: 10
      selector:
        matchLabels:
          app: practicedep
      strategy:
        rollingUpdate:
          maxSurge: 25%
          maxUnavailable: 25%
        type: RollingUpdate
      template:
        metadata:
          labels:
            app: practicedep
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
        app: rollingservice 
      name: rollingservice
      namespace: default
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        app: practicedep
      type: ClusterIP

<Details><summary>Answer</summary>
  
  Firstly(optional step anyways), let us check if given deployment and service work correctly -

      k run temppod --image=nginx:alpine --restart=Never --rm -i -- curl http://rollingservice:80

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
      100   612  100   612    0     0   231k      0 --:--:-- --:--:-- --:--:--  298k
      pod "temppod" deleted

  Now, requirement says rolling update, max 2 pods can go down and additional 2 pods can be created.
  
  Since, we have 10 replicas, we will set maxSurge and maxUnavailable to 20%. Also, you can verify that `type: RollingUpdate`

  We can use `edit` option

      k edit deployment rollingdeployment

  We edit below details(only updated parameters shown)

      strategy:
        rollingUpdate:
          maxSurge: 20%
          maxUnavailable: 20%
      spec:
        containers:
        - image: nginx:1.14.2

  That's it, we are done rooling out deployment. Now, we have been asked to observe the rollout, run below command to do that(pasted output for reference) 

      controlplane $ k rollout status deployment/rollingdeployment
      Waiting for deployment "rollingdeployment" rollout to finish: 9 out of 10 new replicas have been updated...
      Waiting for deployment "rollingdeployment" rollout to finish: 9 out of 10 new replicas have been updated...
      Waiting for deployment "rollingdeployment" rollout to finish: 9 out of 10 new replicas have been updated...
      Waiting for deployment "rollingdeployment" rollout to finish: 9 out of 10 new replicas have been updated...
      Waiting for deployment "rollingdeployment" rollout to finish: 9 out of 10 new replicas have been updated...
      Waiting for deployment "rollingdeployment" rollout to finish: 2 old replicas are pending termination...
      Waiting for deployment "rollingdeployment" rollout to finish: 2 old replicas are pending termination...
      Waiting for deployment "rollingdeployment" rollout to finish: 2 old replicas are pending termination...
      Waiting for deployment "rollingdeployment" rollout to finish: 1 old replicas are pending termination...
      Waiting for deployment "rollingdeployment" rollout to finish: 1 old replicas are pending termination...
      Waiting for deployment "rollingdeployment" rollout to finish: 1 old replicas are pending termination...
      Waiting for deployment "rollingdeployment" rollout to finish: 8 of 10 updated replicas are available...
      Waiting for deployment "rollingdeployment" rollout to finish: 9 of 10 updated replicas are available...
      deployment "rollingdeployment" successfully rolled out

  Now, make sure to retest the service

      k run temppod --image=nginx:alpine --restart=Never --rm -i -- curl http://rollingservice:80

</Details>
