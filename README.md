# Jenkins  for nestjs


```Jenkins
pipeline {
    agent any 
    tools {
        nodejs 'node22' 
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        DOCKER_IMAGE = 'devmius/user-management-service'
        CONTAINER_NAME = 'user-management-service'
        IMAGE_NAME = 'user-management-service'
        PORT = '3001'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'dev', changelog: false, credentialsId: 'coreset_token', poll: false, url: 'https://github.com/coreset/user-management-service.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Run Lint & Tests') {
            steps {
                sh 'npm run lint'
                sh 'npm run test'
            }
        }
        stage('Build Application') {
            steps {
                sh 'npm run build'
            }
        }
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=User-Management-Service  \
                    -Dsonar.sources=src \
                    -Dsonar.projectKey=user-management-service 
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker_hub', toolName: 'docker') {
                        sh "docker build -t $IMAGE_NAME -f Dockerfile ."
                        sh "docker tag $IMAGE_NAME  $DOCKER_IMAGE:latest"
                        sh "docker push $DOCKER_IMAGE:latest"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker_hub', toolName: 'docker') {
                        // Stop and remove any existing container with the same name
                        sh '''
                        if [ "$(docker ps -aq -f name=${CONTAINER_NAME})" ]; then
                            echo "Stopping and removing existing container..."
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                        fi
                        '''
                        // Run the new container
                        sh "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:3000 $DOCKER_IMAGE:latest"
                    }
                }
            }
        }

    }
}
```

### with yarn package (yarn.lock + package-lock.json)    
sometime you you getting package audit issue with package-lock.json file. for that you can convert it to yarn and ignore the package-lock.jason file in following way   
```Jenkins
pipeline {
    agent any 
    tools {
        nodejs 'node22' 
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        DOCKER_IMAGE = 'devmius/user-management-service'
        CONTAINER_NAME = 'user-management-service'
        IMAGE_NAME = 'user-management-service'
        PORT = '3001'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'dev', changelog: false, credentialsId: 'coreset_token', poll: false, url: 'https://github.com/coreset/user-management-service.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install -g yarn'
                sh 'yarn install --frozen-lockfile'
            }
        }
        stage('Run Lint & Tests') {
            steps {
                sh 'yarn lint'
                sh 'yarn test'
            }
        }
        stage('Build Application') {
            steps {
                sh 'yarn build'
            }
        }
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --exclude package-lock.json ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh ''' 
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=User-Management-Service  \
                    -Dsonar.sources=src \
                    -Dsonar.projectKey=user-management-service 
                    '''
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker_hub', toolName: 'docker') {
                        sh "docker build -t $IMAGE_NAME -f Dockerfile ."
                        sh "docker tag $IMAGE_NAME  $DOCKER_IMAGE:latest"
                        sh "docker push $DOCKER_IMAGE:latest"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker_hub', toolName: 'docker') {
                        // Stop and remove any existing container with the same name
                        sh '''
                        if [ "$(docker ps -aq -f name=${CONTAINER_NAME})" ]; then
                            echo "Stopping and removing existing container..."
                            docker stop ${CONTAINER_NAME} || true
                            docker rm ${CONTAINER_NAME} || true
                        fi
                        '''
                        // Run the new container
                        sh "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:3000 $DOCKER_IMAGE:latest"
                    }
                }
            }
        }

    }
}
```

here updated Dockerfile for yarn.  
```Dockerfile
# Use Node.js official image
FROM node:22.14.0

# Set working directory
WORKDIR /app

# Copy package.json and install dependencies
# COPY package*.json ./
COPY package.json ./
COPY yarn.lock ./

# Instal dependencies  
#RUN npm install --only=production
RUN yarn install --frozen-lockfile  

# Ensure NestJS CLI is installed in node_modules
# RUN npm install @nestjs/cli --save-dev
RUN yarn add @nestjs/cli --save-dev

# Copy source files
COPY . .

# Build the project
# RUN npm run build
RUN yarn build

# Expose the application port
EXPOSE 3000

# Start the application
CMD ["node", "dist/main"]
```


