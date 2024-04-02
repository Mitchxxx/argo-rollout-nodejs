pipeline {
    agent any

    options {
        timeout(time: 10, unit: 'MINUTES')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-Mitchel')
        APP_NAME = "mitchxxx/bluegreen-rollout"
    }

    stages {
        stage('SCM Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mitchxxx/argo-rollout-nodejs.git'
            }
        }
        stage('Run Sonarqube') {
            environment {
                scannerHome = tool 'ibt-sonarqube'
            }
            steps {
                withSonarQubeEnv(credentialsId: 'ibt-sonar', installationName: 'IBT sonarqube'){
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Buil docker image') {
            steps {
                sh 'docker build -t $APP_NAME:$BUILD_NUMBER'
            }
        }
        stage('Login to Dockerhub'){
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Trivy Scan Aqua'){
            steps {
                sh 'trivy image $APP_NAME:$BUILD_NUMBER'
            }
        }
        stage('Push image'){
            steps{
                sh 'docker push $APP_NAME:$BUILD_NUMBER'
            }
        }
        stage('Trigger ManifestUpdate'){
            steps {
                build job: 'rollout-manifests', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
            }
        }

    }
    post {
        success {
            slackSend color: '#36a64f', message: "Deployment of myapp to production succeeded!"
        }
        failure {
            slackSend color: '#ff0000', message: "Deployment of myapp to production failed!"
        }
    }
}
