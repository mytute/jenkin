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
