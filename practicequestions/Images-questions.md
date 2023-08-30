# **Images** (use [killerkoda.com](https://killercoda.com/playgrounds/scenario/kubernetes) for practice)

1. Given Dockerfile, build it with name **echosfun**, run with the name **echowell** and check **logs** of the container.
   
   **Dockerfile**
   
       FROM nginx:alpine
       CMD ["sh","-c","echo well"]
   <Details><summary>Answer</summary>

   **Build** (docker build -t < imagename > . )
   
       docker build -t echosfun .

   **List** image to make sure 
   
       docker image ls
   
   **Run** (docker run --name < cotnainername > < imagename >)
   
       docker run --name echowell echosfun
   
   Get **logs** (docker logs <containername>)
   
       docker logs echowell
   </Details>
2. Update the Dockerfile, to echo "hello after update", take build and run the image

   <Details><summary>Answer</summary>

   **Edit** the Dockerfile
   
       FROM nginx:alpine
       CMD ["sh","-c","echo hello after update"]

   _Follow same commands from earlier to take the build, run and check the logs_
   </Details>
3. See all processes(Sample output attached)

   <Details><summary>Answer</summary>
   
       docker ps -a

       CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS                     PORTS     NAMES
       380d0e8567fc   669       "/docker-entrypoint.…"   3 minutes ago   Exited (0) 3 minutes ago             recursing_snyder
   </Details>
4. Remove the image created

   <Details><summary>Answer</summary>

   **Get** the list of images the
   
       docker image ls

   **Delete** the image(docker rmi < imagename > OR < imageID >)
   
       docker rmi echosfun
   </Details>
