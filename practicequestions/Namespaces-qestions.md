# **Namespaces**(use [killerkoda.com](https://killercoda.com/playgrounds/scenario/kubernetes) for practice)

## Please user **alias k=kubectl** to assign kubectl alias to k, You can use kubectl as well

1. List all namespaces

   <Details><summary>Answer</summary>

       k get ns
   </Details>
3. List resources in particular namespace
   
   <Details><summary>Answer</summary>

       k get pods -n default
   </Details>
5. List resources from all namespaces
   
   <Details><summary>Answer</summary>

       k get pods -A
   </Details>
7. Create namespace and set the working namespace to created namespace
   
   <Details><summary>Answer</summary>

   **Create** namespace
   
       k create namespace practiceenough

   **Set** working namespace
   
       k config set-context --current --namespace practiceenough

   Also, we can use alias for the boiler plate string "k config set-context --current --namespace" and set the working namespace.
   Comes handy for setting future namespaces

       alias setns="k config set-context --current --namespace"
       setns practiceenough
   </Details>
