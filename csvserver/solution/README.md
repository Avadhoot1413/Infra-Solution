
### File created by AVADHOOT BHARANKAR ###
###################
###### PART 1 #####
###################
#Installed RHEL 8 on Virtual box
#Installed docker on RHEL VM
sudo yum install docker-ce docker-ce-cli containerd.io
#Started Docker Service
systemctl start docker
#Installed GIT
sudo yum install git
#Executed below commands as mentioned in 
docker pull infracloudio/csvserver:latest
docker pull prom/prometheus:v2.22.0
#Executed command to clone repo
git clone https://github.com/infracloudio/csvserver
#1--> Run the container image infracloudio/csvserver:latest in background and check if it's running.
docker run -d infracloudio/csvserver:latest
#2 --> If it's failing then try to find the reason, once you find the reason, move to the next step.
# we can see the status as Exited (1) , thus we can say that it is failing 
docker ps -a
CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS                     PORTS     NAMES
aa3b577f1ad4   infracloudio/csvserver:latest   "/csvserver/csvserver"   9 seconds ago   Exited (1) 8 seconds ago             stoic_hodgkin
#Checked logs to know reason for failure.
#Reason for failure .there is no /csvserver/inputdata directory. 
[root@localhost solution]# docker logs aa3b577f1ad4
2021/12/02 11:59:58 error while reading the file "/csvserver/inputdata": open /csvserver/inputdata: no such file or directory
#3 --> Write a bash script gencsv.sh to generate a file named inputFile whose content looks like:
#created gencsv.sh that will create file inputFile
vim gencsv.sh
#Content of bash script -->cat gencsv.sh
RANDOM=$$
for i in `seq 10`
do 
	echo "$i," $RANDOM >> inputFile

done

#4 -->Run the container again in the background with file generated in (3) available inside the container (remember the reason you found in (2)).
docker run -d -v /usr/local/Avadhoot/csvserver/inputFile:/csvserver/inputdata infracloudio/csvserver:latest


docker ps
CONTAINER ID   IMAGE                           COMMAND                  CREATED          STATUS          PORTS      NAMES
51589270a5c7   infracloudio/csvserver:latest   "/csvserver/csvserver"   12 minutes ago   Up 12 minutes   9300/tcp   youthful_brattain

#5 --> Get shell access to the container and find the port on which the application is listening. Once done, stop / delete the running container.
docker stop youthful_brattain
youthful_brattain
[root@localhost csvserver]# docker rm youthful_brattain
youthful_brattain

#6 --> Same as (4), run the container and make sure,
# The application is accessible on the host at http://localhost:9393
# Set the environment variable CSVSERVER_BORDER to have value Orange.

docker run -d  --env CSVSERVER_BORDER=Orange -p 9393:9300   -v /usr/local/Avadhoot/csvserver/inputFile:/csvserver/inputdata infracloudio/csvserver:latest
1fd08592aee33855beee9236570a93a17c1f48ec0ae75497322804ac4a5a95a1




####################
##### PART 2 #######
####################

[root@localhost csvserver]# cat docker-compose.yaml 
 version: "3.7"

 services:
   app:
     image: infracloudio/csvserver:latest
     ports:
       - 9393:9300
     volumes:
       - /usr/local/Avadhoot/csvserver/inputFile:/csvserver/inputdata
     environment:
       CSVSERVER_BORDER: Orange


docker-compose up -d
Creating csvserver_app_1 ... done
[root@localhost csvserver]# 
[root@localhost csvserver]# docker ps -a
CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS         PORTS                                       NAMES
580ce643fa74   infracloudio/csvserver:latest   "/csvserver/csvserver"   7 seconds ago   Up 6 seconds   0.0.0.0:9393->9300/tcp, :::9393->9300/tcp   csvserver_app_1
[root@localhost csvserver]# 
[root@localhost csvserver]# 



#####################
######## PART 3 #####
#####################
# 0 --> Delete any containers running from the last part

CONTAINER ID   IMAGE                           COMMAND                  CREATED         STATUS         PORTS                                       NAMES
580ce643fa74   infracloudio/csvserver:latest   "/csvserver/csvserver"   3 minutes ago   Up 3 minutes   0.0.0.0:9393->9300/tcp, :::9393->9300/tcp   csvserver_app_1
[root@localhost csvserver]# docker stop csvserver_app_1 
csvserver_app_1
[root@localhost csvserver]# docker rm csvserver_app_1 
csvserver_app_1
[root@localhost csvserver]# docker ps -a
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@localhost csvserver]# 

#1#2#3#4

[root@localhost csvserver]# cat docker-compose.yaml 
 version: "3"

 services:
   app:
     image: infracloudio/csvserver:latest
     ports:
       - 9393:9300
     volumes:
       - /usr/local/Avadhoot/csvserver/inputFile:/csvserver/inputdata
     environment:
       CSVSERVER_BORDER: Orange

   Prometheus:
     image: prom/prometheus:v2.22.0
     ports:
       - 9090:9090
     volumes:
       - /usr/local/Avadhoot/csvserver/prometheus.yml:/etc/prometheus/prometheus.yml


[root@localhost csvserver]# docker-compose up -d
Creating csvserver_app_1        ... done
Creating csvserver_Prometheus_1 ... done
[root@localhost csvserver]# docker ps -a
CONTAINER ID   IMAGE                           COMMAND                  CREATED              STATUS              PORTS                                       NAMES
bbe9550e956c   prom/prometheus:v2.22.0         "/bin/prometheus --c…"   About a minute ago   Up About a minute   0.0.0.0:9090->9090/tcp, :::9090->9090/tcp   csvserver_Prometheus_1
651c968eaf68   infracloudio/csvserver:latest   "/csvserver/csvserver"   About a minute ago   Up About a minute   0.0.0.0:9393->9300/tcp, :::9393->9300/tcp   csvserver_app_1

