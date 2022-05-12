# Jenkins: How to Build a Java App with Maven
*The following instructions are modified for Windows from the original Jenkins documentation tutorial which you can find [here](https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/) as well as instructions for MacOS.*

### Prerequisites
To follow this guide, you will require:

- Windows machine with:
  - 256 MB of RAM, although more than 2 GB is recommended.
- 10 GB of drive space for Jenkins and your Docker images and containers.
- Docker
- Git 

<br>

### Steps
- [Step 1: Run Jenkins in Docker](#step1)
- [Step 2: Proceed to Jenkins Setup Wizard](#step2)
- [Step 3: Clone This Java App](#step3)


<br>  

## <a name="step1">1. Run Jenkins in Docker (on Windows)</a>
1. Open up a **git bash** terminal, create a directory for the following scripting files called `jenkins-scripts`
2. Make sure Docker is running, then create a **bridge network** using the following docker command:

```docker
docker network create jenkins
```

3. Run the command `nano 1-docker-dind.sh` to create a file which will contain a complicated docker run commmand.  Paste the following within it:

`1-docker-dind.sh`
```sh
#!/bin/bash
docker run --name jenkins-docker --rm --detach \
  --privileged --network jenkins --network-alias docker \
  --env DOCKER_TLS_CERTDIR=/certs \
  --volume jenkins-docker-certs:/certs/client \
  --volume jenkins-data:/var/jenkins_home \
  --publish 3000:3000 --publish 2376:2376 \
  docker:dind
```

> *The above script allows us to execute Docker commands inside Jeknins nodes by building a container we're calling `jenkins-docker`. It does this by **running Docker inside a Docker container.**  Pretty meta, right?*

4. Run the above script in your git bash terminal:

```sh
./1-docker-dind.sh
```

5. Customize the official Jenkins Docker image by executing the following two steps:

  > a. Create a `Dockerfile` within the same `jenkins-scripts` folder and paste in the following content:
  
 ```dockerfile
FROM jenkins/jenkins:2.332.3-jdk11
USER root
RUN apt-get update && apt-get install -y lsb-release
RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
  https://download.docker.com/linux/debian/gpg
RUN echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
  https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
RUN apt-get update && apt-get install -y docker-ce-cli
USER jenkins
RUN jenkins-plugin-cli --plugins "blueocean:1.25.3 docker-workflow:1.28"
 ```

  > b. Build a new Docker image from this Dockerfile and assign the image a meaningful name, e.g `myjenkins-blueocean:2.332.3-1`. These steps will automatically download the official Jenkins Docker image if this hasn't been done before:
  
  ```docker
  docker build -t myjenkins-blueocean:2.332.3-1 .
  ```
  
6. Run your own `myjenkins-blueocean:2.332.3-1` image as a container in Docker using the following docker run command within **CMD** *not a shell script because theres issues with creating the %HOMEDRIVE% volume*.  

Paste the following in Windows CMD and press enter:

```cmd
docker run --name jenkins-blueocean --rm --detach ^
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 ^
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 ^
  --volume jenkins-data:/var/jenkins_home ^
  --volume jenkins-docker-certs:/certs/client:ro ^
  --volume "%HOMEDRIVE%%HOMEPATH%":/home ^
  --publish 8080:8080 --publish 50000:50000 myjenkins-blueocean:2.332.3-1
```

> *This command can be used to re-run the container*

### :tada: Jenkins is up and running!
Go to http://localhost:8080 and you will see the following:

<br>

<img src="/imgs/1-jenkins.png">

<br>

## <a name="step2">2. Proceed to Jenkins Setup Wizard</a>
>*If at any time you need to stop the container run:* `docker stop jenkins-blueocean jenkins-docker`

1. Jenkins is now running in a Docker container exposed at http://localhost:8080.  To retrieve the password run the following command in your terminal:

```
docker logs jenkins-blueocean
```

Copy the password from the container logs - it will look like this:

<img src="/imgs/2-jenkins.png">

<br>

2. Click **Install Suggested Plugins**

3. Creating the first administrator user: When the Create First Admin User page appears, specify your details in the respective fields and click Save and Finish.

When the Jenkins is ready page appears, click Start using Jenkins.

>Notes:
>This page may indicate Jenkins is almost ready! instead and if so, click Restart.
> If the page doesn’t automatically refresh after a minute, use your web browser to refresh the page manually.
> If required, log in to Jenkins with the credentials of the user you just created and you’re ready to start using Jenkins!

<br>

## <a name="step3">3. Clone This Project</a>
1. Clone this repository onto your local machine

## <a name="step4">4. Create your Pipeline Project in Jenkins</a>
1. With Jenkins running, click **create new jobs** under **Welcome to Jenkins!**
> Note: If you don’t see this, click New Item at the top left.

2. In the **Enter an item name** field, specify the name for your new Pipeline project (e.g. `simple-jenkins-demo`).

3. Scroll down and click **Pipeline**, then click **OK** at the end of the page.

4. ( *Optional* ) On the next page, specify a brief description for your Pipeline in the Description field (e.g. An entry-level Pipeline demonstrating how to use Jenkins to build a simple Java application with Maven.)