# **Namespaces**(use [killerkoda.com](https://killercoda.com/playgrounds/scenario/kubernetes) for practice)

## Please user **alias k=kubectl** to assign kubectl alias to k, You can use kubectl as well

1. List all namespaces

   **Ans:**

       k get ns
2. List resources in particular namespace
   
   **Ans:**

       k get pods -n default
3. List resources from all namespaces
   
   **Ans:**

       k get pods -A
4. Create namespace and set the working namespace to created namespace
   
   **Ans:**

   **Create** namespace
   
       k create namespace practiceenough

   **Set** working namespace
   
       k config set-context --current --namespace practiceenough

   Also, we can use alias for the boiler plate string "k config set-context --current --namespace" and set the working namespace.
   Comes handy for setting future namespaces

       alias setns="k config set-context --current --namespace"
       setns practiceenough
