# Deployment questions (use [killerkoda.com](https://killercoda.com/playgrounds/scenario/kubernetes) for practice)

## Please user alias k=kubectl to assign kubectl alias to k, You can use kubectl as well

Q. Create deployment `deploymenque` with image nginx:1.14.1 and 5 replicas. Check the replicaset created for the deployment.

<Details><summary>Answer</summary>

To create deplyment, either you can copy any deployment.yaml from [kubernet](https://kubernetes.io/) site OR use command to generate yaml and edit per requirement.
Run below command to generate deployment in yaml format and store in deployment.yaml. Optionally during exam, you can set alias for boiler plate command `--dry-run=client -oyaml`
More on overall prep [here](https://github.com/letonkargit/CKAD-practice/blob/main/overall-prep-guide.md).

    k create deployment deploymentque --image=nginx:1.14.1 --replicas=5 --dry-run=client -oyaml > deployment.yaml
    
File is store [here](https://raw.githubusercontent.com/letonkargit/CKAD-practice/main/resources/deployment-que1.yaml) for your reference.
You can use `k create -f filename.yaml`. If you have generated file and stored in deployment.yaml 

    k create -f deployment.yaml

OR if just practicing and referring to above mentioned file

    k create -f https://raw.githubusercontent.com/letonkargit/CKAD-practice/main/resources/deployment-que1.yaml

That's all, you have your deployment spun up. You can use `k get deployment -n default` to check the deployment

    k get deployment -n default
    
    NAME            READY   UP-TO-DATE   AVAILABLE   AGE
    deploymentque   5/5     5            5           5m5s

In addition to running the deployment set, we have been asked to check the replicaset(replicaset is created for each deployment), use `k get replicaset -n default`
    
    k get replicaset -n default
    
    NAME                       DESIRED   CURRENT   READY   AGE
    deploymentque-686576f946   5         5         5       5m18s
    
</Details>
