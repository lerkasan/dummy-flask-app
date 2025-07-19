pipeline {
    agent { 
        label 'dind' 
    }

    environment {
        GIT_COMMIT_SHA = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
        GIT_COMMIT_SHORT_SHA = "${GIT_COMMIT_SHA,length=8}"
        IMAGE_NAME = "dummy-flask-app"
        IMAGE_TAG = "${GIT_COMMIT_SHORT_SHA}-${env.BUILD_NUMBER}"
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
            steps {
                sh '''
                cd src
                docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Docker Image') {
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