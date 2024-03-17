![project](https://github.com/MANISANKARDIVI/Board-Game/assets/110891364/5f13d5de-afb3-4f14-b953-c8ed54d63e2e)

###########################################################################################################

prerequisite For Laptop or PC:
``````````````````````````````
Min ram = 8+
Min cpu = I3,  4-core cpu
while using your laptop / pc will behave slow, be patience.


WSL commands:
````````````
$ wsl --install

$ wsl --update

$ wsl --list --online

$ wsl --install -d Ubuntu-22.04



Installation of Java & Maven:
`````````````````````````````
$ sudo apt update -y && sudo apt upgrade -y

$ sudo su -

$ sudo apt install openjdk-17-jre-headless -y

$ sudo apt install maven -y

Docker Installation steps:
`````````````````````````

$ vi dockerscript.sh && chmod +x dockerscript.sh

 ##copy & paste below lines##

sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo usermod -aG docker $USER

sudo chmod 777 /var/run/docker.sock 

systemctl start docker

systemctl enable docker

docker version

Trivy installation steps:-
`````````````````````````
$ vi trivy.sh && chmod +x trivy.sh

####copy and paste below lines####

sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

sudo trivy --version

Jenkins  installation steps:-
```````````````````````
$ vi jenkins.sh && chmod +x jenkins.sh

####copy and paste below lines####

sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
	
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
    /etc/apt/sources.list.d/jenkins.list > /dev/null
	
sudo apt-get update -y
sudo apt-get install jenkins -y

systemctl start jenkins
systemctl enable jenkins

jenkins --version


Install Jenkins plugins:-
`````````````````````````
Eclipse Temurin installer
Maven Integration
Role-based Authorization Strategy
Config File Provider
Pipeline Maven Integration
SonarQube Scanner
Docker
Docker Commons
Docker Pipeline
Docker API
docker-build-step
Kubernetes
Kubernetes Client API
Kubernetes Credentials
Kubernetes CLI
Publish Over SSH
Blue Ocean 
Prometheus metrics
Nexus Artifact Uploader
SSH Agent
Deploy to container
Artifactory
Artifact Deployer


-> restart jenkins server





Declarative Pipeline syntax:-
``````````````````````````````

pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/MANISANKARDIVI/Board-Game.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Code Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Sonar Analysis') {
            steps {
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        
        stage('Trivy scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn install'
            }
        }
        
        stage('nexus artifact') {
            steps {
                script {
                    withMaven(globalMavenSettingsConfig: 'settings.xml', jdk: 'java-17', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                        sh 'mvn clean deploy'
                    }
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    sh "docker build -t manisankardivi/boardgame ."
                }
            }
        }
        
        stage('Docker Image scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html manisankardivi/boardgame'
            }
        }
        
        stage('Upload to Registry') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'registry-cred', variable: 'passwd')]) {
                        sh 'docker login -u manisankardivi -p ${passwd}'
                        sh 'docker push manisankardivi/boardgame'
                    }  
                }
            }
        }
    }
    
    post {
        always {
            script {
                def jobName = env.JOB_NAME
                def buildNumber = env.BUILD_NUMBER
                def pipelineStatus = currentBuild.result ?: 'UNKNOWN'
                def bannerColor = pipelineStatus.toUpperCase() == 'SUCCESS' ? 'green' : 'red'
                def body = """
                    <html>
                    <body>
                    <div style="border: 4px solid ${bannerColor}; padding: 10px;">
                    <h2>${jobName} - Build ${buildNumber}</h2>
                    <div style="background-color: ${bannerColor}; padding: 10px;">
                    <h3 style="color: white;">Pipeline Status: ${pipelineStatus.toUpperCase()}</h3>
                    </div>
                    <p>Check the <a href="${BUILD_URL}">console output</a>.</p>
                    </div>
                    </body>
                    </html>
                """
                emailext (
                    subject: "${jobName} - Build ${buildNumber} - ${pipelineStatus.toUpperCase()}",
                    body: body,
                    to: 'manisankar.divi@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-image-report.html'
                )
            }
        }
    }
}



############################################################################################################################################################################

# BoardgameListingWebApp

## Description

**Board Game Database Full-Stack Web Application.**
This web application displays lists of board games and their reviews. While anyone can view the board game lists and reviews, they are required to log in to add/ edit the board games and their reviews. The 'users' have the authority to add board games to the list and add reviews, and the 'managers' have the authority to edit/ delete the reviews on top of the authorities of users.  

## Technologies

- Java
- Spring Boot
- Amazon Web Services(AWS) EC2
- Thymeleaf
- Thymeleaf Fragments
- HTML5
- CSS
- JavaScript
- Spring MVC
- JDBC
- H2 Database Engine (In-memory)
- JUnit test framework
- Spring Security
- Twitter Bootstrap
- Maven

## Features

- Full-Stack Application
- UI components created with Thymeleaf and styled with Twitter Bootstrap
- Authentication and authorization using Spring Security
  - Authentication by allowing the users to authenticate with a username and password
  - Authorization by granting different permissions based on the roles (non-members, users, and managers)
- Different roles (non-members, users, and managers) with varying levels of permissions
  - Non-members only can see the boardgame lists and reviews
  - Users can add board games and write reviews
  - Managers can edit and delete the reviews
- Deployed the application on AWS EC2
- JUnit test framework for unit testing
- Spring MVC best practices to segregate views, controllers, and database packages
- JDBC for database connectivity and interaction
- CRUD (Create, Read, Update, Delete) operations for managing data in the database
- Schema.sql file to customize the schema and input initial data
- Thymeleaf Fragments to reduce redundancy of repeating HTML elements (head, footer, navigation)

## How to Run

1. Clone the repository
2. Open the project in your IDE of choice
3. Run the application
4. To use initial user data, use the following credentials.
  - username: bugs    |     password: bunny (user role)
  - username: daffy   |     password: duck  (manager role)
5. You can also sign-up as a new user and customize your role to play with the application! 😊
