# **API Deprecation**(use [killerkoda.com](https://killercoda.com/playgrounds/scenario/kubernetes) for practice)

## Please user alias k=kubectl to assign kubectl alias to k, You can use kubectl as well

1. Consider the given deployment.yaml file, observe incorrect apiVersion, get the correct version for deployment resource, fix the deployment.yaml and create the deployment

  deployment.yaml
  
      apiVersion: apps/v2
      kind: Deployment
      metadata:
        labels:
          app: apidepdeploymet
        name: apidepdeploymet
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: apidepdeploymet
        strategy: {}
        template:
          metadata:
            labels:
              app: apidepdeploymet
          spec:
            containers:
            - image: nginx
              name: nginx

<Details><summary>Answer</summary>

  **Get** the group and version information for deployment

      k explain deployment
![image](https://github.com/letonkargit/CKAD-practice/assets/32503981/ab46b26b-b8ed-4dbf-8460-8f7dedb877bc)

  **Correct** the file, change from apiVersion: apps/**v2** to apiVersion: apps/**v1**
  **Create** the deployment

      k create -f deployment.yaml

</Details>
