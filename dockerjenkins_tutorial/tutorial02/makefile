build:
	@docker build -t myjenkins .
run:
	@docker run -p 8080:8080 -p 50000:50000 --name=jenkins-master --mount source=jenkins-log,target=/var/log/jenkins --mount source=jenkins-data,target=/var/jenkins_home -d myjenkins
start:
	@docker start jenkins-master
stop:
	@docker stop jenkins-master
clean:	stop
	@docker rm -v jenkins-master
copy:
	@docker cp jenkins-master:/var/log/jenkins/jenkins.log jenkins.log
passwd:
	@docker exec jenkins-master cat /var/jenkins_home/secrets/initialAdminPassword
clean-data:  clean
	-docker volume rm jenkins-log
	-docker volume rm jenkins-data
create-data:
	@docker volume create jenkins-log
	@docker volume create jenkins-data
