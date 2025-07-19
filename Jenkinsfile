pipeline {
    agent { 
        label 'dind' 
    }

    environment {
        IMAGE_NAME = "dummy-flask-app"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        REGISTRY = "docker.io/lerkasan"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/lerkasan/dummy-flask-app.git',
                    credentialsId: 'github',
                    branch: 'main'
            }
        }

        stage('Build Docker Image') {
            // agent {
            //     docker { image 'docker:28.3.2-dind-rootless' }
            // }
            steps {
                sh '''
                cd src
                docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Docker Image') {
            // agent {
            //     docker { image 'docker:28.3.2-dind-rootless' }
            // }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'REGISTRY_USERNAME', passwordVariable: 'REGISTRY_PASSWORD')]) {
                    sh '''
                    echo "$REGISTRY_PASSWORD" | docker login -u "$REGISTRY_USERNAME" --password-stdin
                    docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }
}