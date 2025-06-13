REVISITING DOCKER & JENKINS

This repo is meant to help me understand:
-> Writing my own Dockerfiles
-> Minimising image dependencies on public images (building own images)
-> Creating and using Data Volumes, including backups (Check linux notes)
-> Creating containerized "Build environments" using containers
-> Handling "secret" data with images and Jenkins

Outcomes:
-> Create Docker images to deploy Jenkins in containers.
-> Quickly spin up Jenkins test environments to test plugins in isolatation.
-> Repproduce cases for problems.
-> Can help test impact of Jenkins upgrades on particular configs.
-> Persist data with volumes and upgrade Jenkins without fear.

DOCKER CHANGES

Docker Volumes
- Persist until manual deletion from docker host.
- Integrate with storage plugins to enable shared data volumes across a cluster.

Docker Networks
- Docker Host uses vm's ip address, networks can be independently created, named and maintained
  without any containers or images.
- Containers can be attached to a network and even the simplest form of ocker offers service discovery
  and exposes all containers inside the network via DNS.

Docker Compose
- Makes is simple to create multiple networks, volumes and services.

---> add rest of changes later!


JENKINS CHANGES

Jenkins version
- Cloudbees Dockerfiles

Docker Plugins

---> more changes later!


PART 1 - Thinking Inside the Container

PART 2 - Puting Jenkins in a Docker Container

 LESSON 1 - Setup and Run First Jenkins Image

 - Deploying Jenkins with these architectural components in mind:
	-> Jenkins master server (Java process)
	-> Jenkins master data (Plugins, Job definitions, etc...)
	-> NGINX web proxy (ssl certs for security etc...)
	-> Build slave agents (Containers that can be ssh'd into, or JNLP connection to, Jenkins Master)

 - DAEMONIZING 
 	-> Running jenkins using the -d flag -- prevents logs from showing on the terminal
		$ docker run -p 8080:8080 --name=jenkins-master -d jenkins/jenkins
	-> Memory settings, using the --env JAVA_OPTS="" flag to limit Jenkins memory usage.
		$ docker run -p 8080:8080 --name=jenkins-master -d --env JAVA_OPTS="-Xmx8192m" jenkins/jenkins
	-> Increasing The Connection Pool using --env JENKINS_OPTS=" --handlerCountMax=300" - To set limit of connections
		$ docker run -p 8080:8080 --name=jenkins-master -d --env JAVA_OPTS="-Xmx8192m" --env JENKINS_OPTS=" --handlerCountMax=300" jenkins/jenkins
 - COMMENTS
   -- The Jenkins container/image is useful but:
	-> Has no consistent logging,
	-> No persistence,
	-> No web server proxy in front of it.
	-> Keep in mind Jenkins versions...
	-> dockerjenkins_tutorial/tutorial01/makefile
 
 LESSON 2 - A JENKINS BASE IMAGE WRAPPER
 
 - Work within a Dockerfile
	-> Set env vars
	-> Create folders and permissions
	-> dockerjenkins_tutorial/tutorial01/Dockerfile
 - Testing the Dockerfile
	-> $ docker build -t myjenkins . -- Builds Jenkins using the Dockerfile
	-> $ docker stop jenkins-master -- If jenkins-master is still running, stop it.
	-> $ docker rm jenkins-master -- Remove the container using this command.
	-> $ docker run -p 8080:8080 --name=jenkins-master -d myjenkins
 - Run Basic Commands Against The Container
	-> docker exec jenkins-master ps -ef |grep java -- This will check if the Jenkins Java process is running.
 - Setup a Log Folder
	-> Add "RUN mkdir /var/log/jenkins" to the Dockerfile to persist the Jenkins log file
	- Copy and view log:
	-> $ docker cp jenkins-master:/var/log/jenkins/jenkins.log jenkins.log; cat jenkins.log;
 - COMMENTS
   -- Seems below params are depricated?
	ENV JAVA_OPTS="-Xmx8192m" -- Works fine with lts jenkins
	ENV JENKINS_OPTS=" --handlerCountMax=300" -- WORKS!


PART 3 - Data That Persists

 - Containers and their data are ephemeral, all the data is lost when container is restarted.
 - Volumes can be used to presserve the data, they sit outside the container and are mounted from the host (Dockerhost).
 - DATA Volumes can also be used to persist data.

 - HOST MOUNTED VOLUMES vs DATA VOLUMES 
        -> Host Mounted Volumes: 
          - Using the Docker host machine filesystems as physical storage that docker containers use to write data to.
          - The data can be network or serially attached to the storage, taking advantage of space and performance.
          - However, the mount points must be pre-configured on the Dockerhost, which eliminates container portabilitu and apps running anywhere.

        -> Data Volumes
          - Whenever a container is created, docker will persist the data in these types of volumes.
          - Docker allows custom volumes that will not be removed when the container is removed.
          - Docker Volumes allow containers to share data without the requirement that the host be configured with a mount point... i.e. no need to touch the host machine.
          - The drawbacks, docker performance is a bit slower, (uses Docker Virtualized File Systems)... !!! performance not noticable for some apps
          - Creating volumes is a bit complex, and because they persist, they must be manually deleted.

 - USING DATA VOLUMES
  -- There are Two Things to persist:
        -> Log files (/var/log/jenkins), and
        -> Jenkins App Data (jobs, plugins, configs...)
        -> Creating the Volume
        $ docker volume create jenkins-data || docker volume create jenkins-log
        $ docker volume ls
        $ docker volume rm jenkins-data || docker volume rm jenkins-log
        $ docker volume prune


PART 4 - JENKINS, DOCKER, PROXIES, AND COMPOSE

 - PROXIES
  -- NGINX - Enforces things like redirects to HTTPS, and masks Jenkins listening on port 8080 with a web server that listens on port 80.

 - Setup Proxy to Jenkins Server
  - When working with multiple Dockerfiles, put them in their own subdir.
	-> e.g. jenkins-nginx & jenkins-master will each have own Dockerfiles.
  - Nginx Dockerfile
	-> Using same centos7, try update later!!!
	-> Install & expose port 80.
  - Nginx Configuration File
	-> To make NGINX not run as a daemon:
		--> daemon off - NGINX will run in the background as a daemon if called from cmd line... this returns exit 0 which causes Docker to think the process has stopped, so it shuts down the container. 
	-> Upping the NGINX worker count to 2:
		--> worker_processes 2; - Tunes NGINX to only create specified individual processes. The number of CPUs allocated is a good guide. 
	-> Event tuning:
		--> use epoll; - This param is a tuning mechanism to use more efficient connection handling models.
		--> accept_mutex off; -  Turned off accept_mutex for speed, because we don’t mind the wasted resources at low connection request counts.
	-> Setting the proxy headers:
		--> proxy_set_header X-Real-IP $remote_addr; - Sets the headers so that Jenkins can interpret the requests properly, which helps eliminate some warnings about improperly set headers.
		--> proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	-> Client sizes:
		--> client_max_body_size 300m; - Helps with file uploads, limits user file uploads to 300m
		--> client_body_buffer_size 128k;
	-> GZIP on:
		--> gzip on; - Turn on gzip compression for speed.
		--> gzip_http_version 1.0;
		--> gzip_comp_level 6;
		--> gzip_min_length 0;
		--> gzip_buffers 16 8k;
		--> gzip_proxied any;
		--> gzip_types text/plain text/css text/xml text/javascript application/xml application/xml+rss application/javascript application/json;
		--> gzip_disable "MSIE [1-6]\.";
		--> gzip_vary on;

  - Jenkins Configuration File For NGINX (see https://wiki.jenkins-ci.org/display/JENKINS/Jenkins+behind+an+NGinX+reverse+proxy for more info)
	-> proxy_pass http://jenkins-master:8080; - Expects a domain name of jenkins-master to exist, which will come from Docker networks.  This must be referenced whenever Docker networks are not in use.
		- Cannot be srt to localhost, it will see that as the NGINX container host.
		- To avoid using Docker networks, it'd have to point to the IP address of the Dockerhost.
		- Docker networks makes this much easier (https://docs.docker.com/engine/network/)

  - Docker Network
	-> $ docker network create --driver bridge jenkins-net - Creates docker network
	  --> Creates the network for the 2 containers to find each other.  
	  --> Docker networks uses "Automatic Service Delivery", which creates dns names on the network that match the names of the containers.
	  

 - DOCKER COMPOSE & JENKINS
   - Docker compose is "A tool designed for running complex apps with Docker" - (https://docs.docker.com/compose/)
   - Compose handles building images and maintaining responsibility around what to stop and start when the application is rerun.
   - Can also help create data volumes and networks


PART 5 - TAKING CONTROL OF DOCKER IMAGE
 - Breakdown the image and understand how it really works.
 - Manage dependencies by understanding where the FROM clause in the Dockerfile points and what it is retrieving
 - BENEFITS:
	-> Control the OS layer.
	-> Limit secutity risks.

 - DISCOVERING DEPENDENCIES

  - Using the FROM jenkins/jenkins:2.112,
	-> Go to dockerhub and see how it is built inside the Dockerfile
	-> Take note for Jenkins in the Dockerfile: 
		1. Environment variables : JENKINS_HOME, JENKINS_SLAVE_PORT, JENKINS_UC, and JENKINS_VERSION.
		2. Dockerfile ARG (build time arguments) are set up for:
		   -> user
		   -> group
		   -> uid
		   -> gid
		   -> http_port
		   -> agent_port
		   -> TINI_VERSION
		   -> JENKINS_VERSION
		   -> JENKINS_SHA
		   -> JENKINS_URL
		3. The image uses Tini to help manage any zombie processes, for Jenkins it is necessary.
		4. The Jenkins war file is pulled into the image by the Dockerfile with a curl request.
		5. The file itself installs curl, and git using apt-get, therefore the OS must be Debian/Ubuntu flavor of Linux.
		6. Files copied into the container from source: 
		   -> jenkins.sh - A shell script that starts Jenkins using the JAVA_OPTS and JENKINS_OPTS environment variables we set.
		   -> plugins.sh - A legacy version of install-plugins.sh and is deprecated. 
		   -> init.groovy - Runs when Jenkins starts, and as a groovy file it runs with context inside the Java WAR that is Jenkins. In this case it’s taking the environment variable set for the Slave agents (50000) and making sure that is the port Jenkins uses.
		   -> Install-plugins.sh - This is a handy file that can be run to auto-download a list of plugins from a plugins text file.
		7. Exposed Ports are:
		   -> http_port - 8080, for Jenkins to listen on, and
		   -> agent_port -  50000 Slaves to talk to Jenkins.
   - Using the FROM openjdk:8-jdk

 - CREATING CUSTOM DOKERFILE
   -> See tutorial06/Dockerfile


PART 6 - BUILDING WITH JENKINS INSIDE EPHEMERAL DOCKER CONTAINERS

  - DOCKER EXECUTION MODEL
	-> This model assumes the slave is the Dockerhost, and is treated as a physical machine... 
	-> When a jenkins job starts, it creates/syncs a working directory on the slave directory, leverages the "docker run" and "docker exec" commands to spin up containers, then mounts its local workspace inside
	-> The container is a virtual scratchpad and isolated environment.
	-> When the build job completes, all the binaries and build artifacts it produced will be n the slave in the traditional jenkins workspace. Jenkins then shuts down and does post-build cleanups.

  - DOCKER EPHEMERAL SLAVE MODEL
	-> This model aims to leverage the autonomous and isolated nature of docker containers to scale a Jenkins build farm to meet any demand placed upon it.
	-> Instead of the traditional array of pre-allocated slaves or at-the-ready vm's, this model treats the entire container itself as a slave.
	-> Jenkins will spin up containers whenever there's a demand for execution, automatically configure Jenkins to accept the containers as new slaves, execute the jobs, and finally shut down the container and deallocate the slave

  - LESSONS LEARNED

	-> Operating Dockerhosts at a scale is not simple
	  - Disk space - Docker images eat space, every running container eats space, sometimes dead containers eat space... therefore monitoring disk space is essential.
	
	-> Cleaning up images and containers is like garbage collection
	  - Everytime a new image is pushed to Dockerhost, it leaves "dangling" unused layers behind... clean them up to avoid disk space issues.

	-> Adding new images/slaves to a fleet of Dockerhosts is time consuming
	
	-> MOnitoring is Essential
	
	-> Jenkins auditing can bite you
	

PART 7 - BUILDING WITH JENKINS INSIDE EPHEMERAL DOCKER CONTAINERS (PRACTICAL)

--- Completed tutorial... Check jenkins-prod repo for up to date prod ready Jenkins
