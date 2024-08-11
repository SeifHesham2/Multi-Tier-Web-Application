pipeline {
    environment {
        NodejsHome = tool "myNode"
        dockerHome = tool "myDocker"
        SonarQubeHome = tool "mySonar"
        OWASP_HOME = tool "DP-Check"
        TMDB_V3_API_KEY = credentials('TMDB_V3_API_KEY')
        SONARQUBE_TOKEN = credentials('SonarNetflix')
        PATH = "${dockerHome}/bin:${NodejsHome}/bin:${SonarQubeHome}/bin:${OWASP_HOME}/bin:${env.PATH}"
    }
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out...'
                checkout scm
            }
        }
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'npm install'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker Image...'
                    def imageTag = "${env.BUILD_NUMBER}"
                    dockerImage = docker.build("seifseddik120/my-app:${imageTag}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    echo 'Pushing Docker Image...'
                    docker.withRegistry('https://index.docker.io/v1/', 'DockerHub') {
                        dockerImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        stage('Update Kubernetes Manifests') {
            steps {
                echo 'Updating Kubernetes manifests...'
                sh """
                sed -i 's|image: seifseddik120/my-app:.*|image: seifseddik120/my-app:${env.BUILD_NUMBER}|g' kubernetes/application.yaml
                """
                sh 'git add kubernetes/application.yaml'
                sh 'git commit -m "Update image tag to build number ${env.BUILD_NUMBER}"'
                sh 'git push origin HEAD:master'
            }
        }
        stage('Argo CD Sync') {
            steps {
                echo 'Triggering Argo CD sync...'
                sh 'argocd app sync my-app' 
            }
        }
    }
}
