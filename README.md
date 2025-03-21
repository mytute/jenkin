# Jenkins  

CI/CD :   
CI is a way which multiple people can work on the same Project without effecting each other work.  
when we deploy the application to server that is CD.   

why Jenkins is perfer to use   
1. Jenkins is free
2. Jenkins is very easy to use
3. Jenkins plugins(more thant 1000)

run on server   
```bash
$ sudo apt update
$ sudo apt install openjdk-11-jre
$ java --version  
```

install jenkins on server   
```bash
$ sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
$ echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install jenkins
```

start Jenkins   
```bash
$ sudo systemctl enable jenkins   
$ sudo systemctl start jenkins  
$ sudo systemctl status jenkins  
```

debug  
```bash  
# Check if another service is using this port:  
$ sudo netstat -tulnp | grep 8080
# check which process is using port 8080:  
$ sudo lsof -i :8080
# If the process is not critical, stop it:
$ sudo kill -9 <PID>
# restart jenkins   
$ sudo systemctl restart jenkins
```

features on jenkins    
Dashboard>Manage Jenkins>Configure System  : pleace to configure new server (like sonar-cube)   
Dashboard>Manage Jenkins>Global Tool Configuration  : pleace to configure new tools (like jdk )   
Dashboard>Manage Jenkins>Manage Plugins  : pleace to add/remove/update plugins and after added it will showing "Global Tool Configuration" section.      
Dashboard>Manage Jenkins>Global Tool Configuration  : pleace to configure new tools (like jdk )   
Dashboard>Manage Jenkins>Manage Nodes and Clouds  :      


install following plugins   
1. Eclipse Temurin installer by searching 'JDK'
2. Docker by searching 'docker'
3.  OWASP Dependency-Check  by searching 'OWASP'   


```bash
sudo apt update
sudo apt install maven -y
mvn -version
```

after install docker into vps   
```bash
$ suoo systemctl restart docker
$ sudo usermod -aG docker $USER
$ sudo newgrp docker
$ sudo chmod 666 /var/run/docker.sock
$ sudo systemctl restart docker  
```

run sonar-qube locally   
```bash
$ docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

Jenkins script   
```Jenkins
pipeline {
    agent any 
    tools {
        jdk 'jdk'
        maven 'maven'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'coreset_token', poll: false, url: 'https://github.com/jaiswaladi246/Ekart.git'
            }
        }
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
    }
}
```

```bash
pipeline {
    agent any 
    tools {
        jdk 'jdk'
        maven 'maven'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'coreset_token', poll: false, url: 'https://github.com/jaiswaladi246/Ekart.git'
            }
        }
        stage('COMPILE') {
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=Shopping-Cart \
                    -Dsonar.java.binaries=target/classes \
                    -Dsonar.projectKey=Shopping-Cart
                    '''
                }
            }
        }

    }
}
```

now build application with Docker   
first you need to create docker access tokens   
Go to Jenkins Dashboard → Manage Jenkins → Manage Plugins.
Search for "Docker Pipeline" and install/update it.
Restart Jenkins after installation.   

Your Jenkins pipeline must first build the JAR before attempting to copy it into the Docker image.
```Jenkins
stage('Build') {
    steps {
        sh 'mvn clean package -DskipTests'
    }
}

stage('Build Docker Image') {
    steps {
        sh 'docker build -t shopping-cart -f docker/Dockerfile .'
    }
}

```


