pipeline {
    agent any
    tools {
        jdk 'java'
        maven 'maven'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('gitclone') {
            steps {
                git 'https://github.com/jetty251/fullstack-blogging-app.git'
            }
        }
        stage('compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('trivy fs scan') {
            steps {
                sh 'trivy fs --format table -o fs.html .'
            }
        }
        stage('sonarqube') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh '$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=blogging -Dsonar.projectKey=blogging -Dsonar.java.binaries=target'
                    }
                }
            }
        }
        stage('build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('s3') {
            steps {
                sh 'aws s3 cp /var/lib/jenkins/workspace/blogging-app/ s3://jetty-hotstar/blogging-app/ --recursive'
            }
        }
        stage('docker build image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-crd', toolName: 'docker') {
                        sh 'docker build -t jetty45/blogging:latest .'
                    }
                }
            }
        }
        stage('trivy image scan') {
            steps {
                sh 'trivy image jetty45/blogging:latest'
            }
        }
        stage('docker push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-crd', toolName: 'docker') {
                        sh 'docker push jetty45/blogging:latest'
                    }
                }
            }
        }
        stage('kubernetes-deploy') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://4E037BAC003EE9C5A8A0653A4E339A4C.gr7.us-east-1.eks.amazonaws.com') {
                        sh 'kubectl apply -f deployment-service.yml'
                        sleep 20
                    }
                    
                }
            }
        }
        stage('kubernetes-verify') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'devopsshack-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://4E037BAC003EE9C5A8A0653A4E339A4C.gr7.us-east-1.eks.amazonaws.com') {
                        sh 'kubectl get pods'
                        sh 'kubectl get svc'
                    }
                    
                }
            }
        }
        
    }
}
