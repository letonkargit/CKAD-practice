# Deployment strategies questions (use [killerkoda.com](https://killercoda.com/playgrounds/scenario/kubernetes) for practice)

## Please user alias k=kubectl to assign kubectl alias to k, You can use kubectl as well

Q. Given deployment **practicedep** with 10 replicas in **default** namespace, running with **nginx:1.14.1**. Task is to update image to **nginx:1.14.2** using canary strategy.
   Run maximum of 10 pods(50% pods be in new deployment with image nginx:1.14.2 and 50% in old deployment). Service is exposed for testing(attached yaml)

   
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

  **Ans:**
  
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

  Now, requirement says **canary** update, 50% to new deployment and 50% to old deployment. Canary is about running multiple versions of app that handles the traffic.
  Read more about canary-deployments [here](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#canary-deployment).
  Services work based on the label selectors, thus, creating new deployment with expected version and adding labels those can be matched by service would suffice for this question.
  In addition to that we will make sure only expected number of pods per deployment are running.
  So, we can follow - keep new deployment ready, scale down existing deployment to 5(50% in this case), create new deployment with 5 replicas and test.

  keep new deployment ready, new-deployment.yaml

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      labels:
        app: newrollingdeployment
      name: newrollingdeployment
      namespace: default
    spec:
      replicas: 5
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
          - image: nginx:1.14.2
            imagePullPolicy: IfNotPresent
            name: nginx
          restartPolicy: Always

   Scale down existing deployment to 5 pods - this can either be done using `scale` option or update the deployment.yaml.
   Handy way to scale down using `scale` option

         controlplane $ k get deployments
         NAME                READY   UP-TO-DATE   AVAILABLE   AGE
         rollingdeployment   10/10   10           10          2m2s
         
         controlplane $ k scale deployment rollingdeployment --replicas=5
         deployment.apps/rollingdeployment scaled
         
         controlplane $ k get deployments
         NAME                READY   UP-TO-DATE   AVAILABLE   AGE
         rollingdeployment   5/5     5            5           2m22s

   Lastly, create deployment with new image

         controlplane $ k create -f new-deployment.yaml 
         deployment.apps/newrollingdeployment created
         
         controlplane $ k get deployments
         NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
         newrollingdeployment   5/5     5            5           4s
         rollingdeployment      5/5     5            5           3m5s

   That's it! Feel free to test using same command we used earlier, service would start sending traffic to both the deployments

         k run temppod --image=nginx:alpine --restart=Never --rm -i -- curl http://rollingservice:80

   ## For understanding purpose 

   If you check the labels on the deployment, you will see they are labeled different than what labels service matches(Selector: app=practicedep)

         controlplane $ k get deployments --show-labels
         NAME                   READY   UP-TO-DATE   AVAILABLE   AGE    LABELS
         newrollingdeployment   5/5     5            5           63s    app=newrollingdeployment
         rollingdeployment      5/5     5            5           4m4s   app=rollingdeployment

         controlplane $ k describe  svc rollingservice
         Name:              rollingservice
         Namespace:         default
         Labels:            app=rollingservice
         Annotations:       <none>
         Selector:          app=practicedep
         Type:              ClusterIP
         IP Family Policy:  SingleStack
         IP Families:       IPv4
         IP:                10.103.187.88
         IPs:               10.103.187.88
         Port:              <unset>  80/TCP
         TargetPort:        80/TCP
         Endpoints:         192.168.1.11:80,192.168.1.12:80,192.168.1.24:80 + 7 more...
         Session Affinity:  None
         Events:            <none>

   It's because, it matches labels on the pods

         controlplane $ k get pods --show-labels
         NAME                                    READY   STATUS    RESTARTS   AGE     LABELS
         newrollingdeployment-6d8cbb7d59-fcrrv   1/1     Running   0          4m12s   app=practicedep,pod-template-hash=6d8cbb7d59
         newrollingdeployment-6d8cbb7d59-gscgn   1/1     Running   0          4m12s   app=practicedep,pod-template-hash=6d8cbb7d59
         newrollingdeployment-6d8cbb7d59-s5wph   1/1     Running   0          4m12s   app=practicedep,pod-template-hash=6d8cbb7d59
         newrollingdeployment-6d8cbb7d59-vtf9b   1/1     Running   0          4m12s   app=practicedep,pod-template-hash=6d8cbb7d59
         newrollingdeployment-6d8cbb7d59-vz87h   1/1     Running   0          4m12s   app=practicedep,pod-template-hash=6d8cbb7d59
         rollingdeployment-67bdf44b4-g4zbh       1/1     Running   0          7m13s   app=practicedep,pod-template-hash=67bdf44b4
         rollingdeployment-67bdf44b4-g9kmt       1/1     Running   0          7m13s   app=practicedep,pod-template-hash=67bdf44b4
         rollingdeployment-67bdf44b4-k989f       1/1     Running   0          7m13s   app=practicedep,pod-template-hash=67bdf44b4
         rollingdeployment-67bdf44b4-np57s       1/1     Running   0          7m13s   app=practicedep,pod-template-hash=67bdf44b4
         rollingdeployment-67bdf44b4-tsjgh       1/1     Running   0          7m13s   app=practicedep,pod-template-hash=67bdf44b4
