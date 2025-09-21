pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('amonkincloud-dockerhub')
    }
    stages {
        stage('Build docker image') {
            steps {
                sh 'docker build -t ylmt/flaskapp:$BUILD_NUMBER .'
            }
        }
        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push image') {
            steps {
                sh 'docker push ylmt/flaskapp:$BUILD_NUMBER'
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
