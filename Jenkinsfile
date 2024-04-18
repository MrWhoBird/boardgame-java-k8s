pipeline {
    agent any
    
    tools{
        jdk 'jdk17'
        maven 'maven3'
        dockerTool 'docker'
    }
    
    environment{
        SCANNER_HOME = tool 'sonarScanner'
    }
    
    stages {
        stage('Git checkout') {
            steps {
                git 'https://github.com/MrWhoBird/boardgame-java-k8s.git'
            }
        }
        stage('Maven compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Maven test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('File system scan with trivy') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('Maven build') {
            steps {
                sh 'mvn package'
            }
        }
        stage('Deploy to nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        stage('Build and tag docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-login', toolName: 'docker') {
                        sh 'docker build -t devopst/boardgame:latest .'
                    }
                }
            }
        }
        stage('Image scan with trivy') {
            steps {
                sh 'trivy image --format table -o trivy-fs-report.html devopst/boardgame:latest'
            }
        }
        stage('Push docker images to hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-login', toolName: 'docker') {
                        sh 'docker push devopst/boardgame:latest'
                }
                }
            }
        }
    }
}
