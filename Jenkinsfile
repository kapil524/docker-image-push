pipeline {
    agent any
    environment {
        DOCKERHUB = credentials('kmanwani-dockerhub')   // your credential ID
        IMAGE_NAME = "flask-image"
        REPO = "kmanwani"   // your Docker Hub username
    }
    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REPO/$IMAGE_NAME:$BUILD_NUMBER .'
            }
        }
        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
            }
        }
        stage('Push Docker Image') {
            steps {
                sh 'docker push $REPO/$IMAGE_NAME:$BUILD_NUMBER'
            }
        }
    }
    post {
        always {
            script {
                sh 'docker logout'
            }
        }
    }
}
