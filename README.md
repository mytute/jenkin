# jenkin
jenkin tutorial (https://www.youtube.com/watch?v=soIpt3c7tVE)   

run following command   
$ docker run hello-world   

get Dockerfile for create jenkins image   
$ mkdir -p ~/repo  && cd repo && git clone git@github.com:vdespa/install-jenkins-docker.git     
>Dockerfile  
```bash
FROM jenkins/jenkins:lts-jdk17
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
```

> docker-compose.yml   
```bash
services:
  jenkins-docker:
    image: docker:dind
    container_name: jenkins-docker
    privileged: true
    environment:
      - DOCKER_TLS_CERTDIR=/certs
    volumes:
      - jenkins-docker-certs:/certs/client
      - jenkins-data:/var/jenkins_home
    ports:
      - "2376:2376"
    networks:
      jenkins:
        aliases:
          - docker
    command: --storage-driver overlay2

  my-jenkins:
    image: my-jenkins
    build:
      context: .
    container_name: my-jenkins
    restart: on-failure
    environment:
      - DOCKER_HOST=tcp://docker:2376
      - DOCKER_CERT_PATH=/certs/client
      - DOCKER_TLS_VERIFY=1
    volumes:
      - jenkins-data:/var/jenkins_home
      - jenkins-docker-certs:/certs/client:ro
    ports:
      - "8080:8080"
      - "50000:50000"
    networks:
      - jenkins

networks:
  jenkins:
    driver: bridge

volumes:
  jenkins-docker-certs:
  jenkins-data:
```

to build Jenking Docker image  
```bash
$ sudo docker build -t my-jenkin .
```

to start Jenkins  
```bash
$ sudo docker compose up -d  
```

unlock jenkins   
```bash
# in termimal
$ sudo docker exec -it my-jenkins bash
jenkins@b3036cb40f09:/$ cat /var/jenkins_home/secrets/initialAdminPassword
# result :  
# df8bfc43dfgdgcb47a79e1235468785
# goto localhost:8080 && enter above password  
# click "install suggested plugins"
# create profile with password 
# keep "Jenkins URL" as it is  
```

create first Jenkins job   
```bash
(click) "+ New Item" on dashboard menu > (input) item name > (select) "Freestyle project" && (click) "OK" button > 
(click) "Build Steps" on "Configure" menu > (select) "Execute shell"  > (input) command as 'echo "hello from Jenkins"' > 
(click) "Save" button > (goto) "Dashboard" > (select) newly created item > (click) "Build Now" form menu for run the command. >
(click) build from "Build History" from the left menu > (click) "Console Output" from the left menu   
```



