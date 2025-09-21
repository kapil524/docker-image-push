pipeline {
    agent any
    environment {
        DOCKERHUB = credentials('kmanwani-dockerhub')   // Docker Hub credentials
        IMAGE_NAME = "flask-image"
        REPO = "kmanwani"   // Docker Hub username
        CONTAINER_NAME = "flask-app"  // Name of container on server
        PORT = "80"                  // Port to expose
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
        stage('Run Docker Container') {
            steps {
                // Stop existing container (if any) and remove it
                sh '''
                if [ $(docker ps -a -q -f name=$CONTAINER_NAME) ]; then
                    docker stop $CONTAINER_NAME
                    docker rm $CONTAINER_NAME
                fi
                '''
                // Run new container on port 80
                sh 'docker run -d --name $CONTAINER_NAME -p $PORT:8080 $REPO/$IMAGE_NAME:$BUILD_NUMBER'
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}
