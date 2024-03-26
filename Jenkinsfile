pipeline {
    agent any

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/joannapauline/Board-Game-dasi.git'
            }
        }

        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('sonarqube analysis') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar2') {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('sonarqube-quality Gate') {
            steps {
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar2'
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Nexux deploy') {
            steps {
                script {
                    withMaven(globalMavenSettingsConfig: '6500f2d5-f14d-468d-9d6e-01a83e127482', jdk: 'java', maven: 'maven', mavenSettingsConfig: '', traceability: true) {
                        sh 'mvn clean deploy'
                    }
                }

            }
        }

        stage('Docker Image Build & Tag') {
            steps {
                sh 'docker build -t dasivardhani/newproj .'
            }
        }

        stage('Docker Registry') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-login', toolName: 'docker') {
                        sh 'docker push dasivardhani/newproj'
                    }
                }
            }
        
        }
        stage('Kubernetes Deploy') {
            steps {
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.38.58:6443') {
                        sh 'kubectl apply -f depolyment-service.yaml -n webapps'
                        sh 'kubectl get svc -n webapps'
       
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
                    to: 'dasivardhani@gmail.com',
                    from: 'jenkins@example.com',
                    replyTo: 'jenkins@example.com',
                    mimeType: 'text/html',
                    attachmentsPattern: 'trivy-fs-report.html,trivy-image-report.html '
                )
            }
       }
    
    }
    
    

}
