
# Docker Donegal - Ninjas Belt 
### Docker DOJO (Where Ninjas are born!) 
- Pipeline - Happy Path :)
![](images/pipeline-happy-path.png)


## Fork our project to your github accout

- Go to **https://github.com/dockerdonegal/ci-cd-dojo** and click on the **Fork** button in the top right corner. 

![fork](images/1-fork-dd-ci-cd-dojo.png)

- This will take only a few seconds, browser response "Forking dockerdonegal/ci-cd-dojo". 
Now you should have our project in your own repo. From now on we will be using your newly forked project. 
- Fork and Enjoy ;)

- **https://github.com/{{your-repo-username}}/ci-cd-dojo**
## Clone your project
```
# Clone your project down to your dev machine
git clone git@github.com:Gmanweb/ci-cd-dojo.git

# change into the following directory
cd ci-cd-dojo/devops/

# list repo branches
git branch -a

# Checkout develope branch
git checkout develop
```

## Build your docker image
- To build your jenkins image, run the following command replacing ` {{dockerhub-username}}` with your DockerHub username.
- (This may take a while depending on your network... So make sure your jenkins/Jenkinsfile is correct) 
```
# Sample 
docker build --no-cache --file ./jenkins/Dockerfile -t {{dockerhub-username}}/dd-jenkins:latest ./jenkins
# Example
docker build --no-cache --file ./jenkins/Dockerfile -t gmanweb/dd-jenkins:latest ./jenkins
```
- Terminal Response
```
Sending build context to Docker daemon   2.56kB
Step 1/9 : FROM jenkins/jenkins:2.127-alpine
2.127-alpine: Pulling from jenkins/jenkins
ff3a5c916c92: Already exists 
a8906544047d: Pull complete
... 
...
Successfully built 5f8c98ab3d70
Successfully tagged gmanweb/dd-jenkins:latest

```
## Login into Docker
- Run `docker login` from the terminal, you will be promted for your username and password.
```
docker login
```
Example:\
![](images/2.docker-login.png)

## Push to DockerHub
- (This may take a while too... Be patient and make sure you're pushing to the right DockerHub account - yours)

```
# Sample 
docker push {{dockerhub-username}}/dd-jenkins:latest
# Example
docker push gmanweb/dd-jenkins:latest
```

- You should expect a similar result:
```
The push refers to repository [docker.io/gmanweb/dd-jenkins]
b9d438413ac4: Pushing [========================================>          ]   93.9MB/117.2MB
...
...
cd7100a72410: Layer already exists 
latest: digest: sha256:d1ddc99d5d9c357906be4ba962ae15f0f02702e50d3e779e380819cfe9cc5361 size: 4081

```

## Initialize a swarm
```
docker swarm init
```

## Create Network

> **Prerequisites:**
> 
> - Firewall rules for Docker daemons using overlay networks
> 
> You need the following ports open to traffic to and from each Docker host participating on an overlay network:
> 
> 
> - TCP port 2377 for cluster management communications
> - TCP and UDP port 7946 for communication among nodes
> - UDP port 4789 for overlay network traffic
> 
>Before you can create an overlay network, you need to either initialize your Docker daemon as a swarm manager using docker swarm init or join it to an existing swarm using docker swarm join. Either of these creates the default ingress overlay network which is used by swarm services by default. You need to do this even if you never plan to use swarm services. Afterward, you can create additional user-defined overlay networks


- To create an overlay network which can be used by swarm services or standalone containers to communicate with other standalone containers running on other Docker daemons, add the --attachable flag:
```
docker network create --driver overlay --attachable --subnet=10.0.0.0/16 dd-network
```
- You can specify the IP address range, subnet, gateway, and other options. See docker network create --help for details.



## Pull the images
Pull the image associated with your service defined in your docker-compose.yml
```
docker-compose pull
```

## Deploy stack
Run the following command to deploy your stack. 
```
docker stack deploy --compose-file docker-compose.yml ddninja
```
Terminal Response
```
Creating service ddninja_jenkins
```
TODO: Image in Compose FIle

### Check services
docker service ls
```
ID                  NAME                MODE                REPLICAS            IMAGE                      PORTS
gkykm51ddsqo        ddninja_jenkins     replicated          1/1                 javapi/dd-jenkins:latest   *:8080->8080/tcp
```

### Jenkins logs
Open a seperate terminal/cmd and run the following command to tail jenkins container logs
```
docker logs ddninja_jenkins.1.xxxxxxxxxxxxxxxxxx -f
```

## Configure your Jenkins Server
Open your browser at `http://127.0.0.1:8080/`

### Add Credential
- We will be adding your **DockerHub** and **GitHub** login details to jenkins.
> NOTE: Very Important when adding your credentials to use the following **IDs:**
> * github = githubid
> * dockerhub = dockerhubid


- From the screen presented, click on Credential -> global -> add credentails. 
We will be adding your credentials for (github, dockerhub)

**Credentials:**

![](images/credentials.png)

**Global:**

![](images/credential-global.png)

**Add credentails:**

![](images/credential-add.png)

**Github credentails:**

![](images/credential-github.png)

**Dockerhub credentails:**

![](images/credential-dockerhub.png)

**Your 2 (dockerhub, Github) credentails:**
- Remember the ID for each is important.

![](images/credential-all.png)

### Create your pipeline JOB
- Now we can create our first pipeline job.

- Back to Jenkins Home click on **New Item**.


![](images/credentials.png)


- Then create a new **multibranch Pipeline**:

![](images/dd-create-new-pipeline.png)


- Set up your Jenkins build...

![](images/job-2.png)
![](images/job-3.png)
![](images/job-4.png)

### Run your JOB (Wait it to be scanned and run automatically)
- First run will fail because of build parameters...
- Second and subsequent runs should be investigated - watch the logs ;)

- When everything is correct, you should see the pipeline going GREEN =)
![](images/3.jenkins-pipline-sample.png)

### Tips
- Check if the Jenkinsfile is consistent
- CHeck if the Dockerfile is consistent
- Check if the jenkins/Dockerfile is consistent
- Is the CI project consistent? ;)

### Clean up - Delete Stack
```
docker stack rm $(docker stack ls --format '{{.Name}}')
docker container prune -f

# If there's any incorrect behaviour related to an image (volumes)
docker system prune --volumes
# Remove QA and PROD Containers
docker container rm -f ninja-belt-prod ninja-belt-qa
```


### Contact us!
> 
> Ping us on slack: **dockerdonegal.slack.com**
> 
> Follow us on Twitter: **@dockerdonegal**
> 

- Follow documentation of jenkins (pipelines) at https://jenkins.io/doc/book/pipeline/docker/

![](images/dd-devops-resized.png)
