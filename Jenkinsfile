pipeline {
    agent { 
        label 'dind' 
    }

    // agent none

    environment {
        GIT_COMMIT_SHA = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true).trim()
        GIT_COMMIT_SHORT_SHA = sh (script: "git rev-parse --short HEAD", returnStdout: true).trim()
        IMAGE_NAME = "dummy-flask-app"
        IMAGE_TAG = "${GIT_COMMIT_SHORT_SHA}-${env.BUILD_NUMBER}"
        REGISTRY = "docker.io/lerkasan"
    }

    stages {
        stage('Checkout') {
            // agent { 
            //     label 'dind' 
            // }
            steps {
                git url: 'https://github.com/lerkasan/dummy-flask-app.git',
                    credentialsId: 'github',
                    branch: 'main'
            }
        }

        stage('Test') {
            // agent { 
            //     label 'python' 
            // }
            // agent {
            //     docker {
            //         image 'python:3.13-alpine3.22'
            //         label 'dind'
            //         // args  '-v /tmp:/tmp'
            //     }
            // }

            steps {
                container('python') {    
                    sh '''
                    pip3 install -r requirements.txt
                    pytest tests/ --doctest-modules --junitxml=test-results.xml
                    coverage run -m pytest
                    coverage xml
                    '''

                    junit 'test-results.xml'

                    recordCoverage(tools: [[parser: 'JACOCO']],
                    id: 'jacoco', name: 'Coverage',
                    sourceCodeRetention: 'EVERY_BUILD',
                    sourceDirectories: [[path: 'src']],
                    qualityGates: [
                        [threshold: 60.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]
                    ])
                }
            }
        }    

        stage('Build Docker Image') {
            // agent { 
            //     label 'dind' 
            // }
            steps {
                sh '''
                cd src
                sleep 30
                docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Push Docker Image') {
            // agent { 
            //     label 'dind' 
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


recordCoverage(tools: [[parser: 'JACOCO']],
        id: 'jacoco', name: 'JaCoCo Coverage',
        sourceCodeRetention: 'EVERY_BUILD',
        qualityGates: [
                [threshold: 60.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
                [threshold: 60.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]])


// https://medium.com/geekculture/jenkins-pipeline-python-and-docker-altogether-442d38119484