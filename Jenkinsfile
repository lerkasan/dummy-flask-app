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
            agent { 
                label 'python' 
            }
            steps {
                git url: 'https://github.com/lerkasan/dummy-flask-app.git',
                    credentialsId: 'github',
                    branch: 'main'
            }
        }

        stage('Test') {
            agent { 
                label 'python' 
            }

            steps {
                // container('python') {    
                    sh '''
                    pip3 install -r src/requirements.txt
                    pytest tests/ --doctest-modules --junitxml=test-results.xml
                    coverage run -m pytest
                    coverage xml
                    '''

                    junit 'test-results.xml'

                    // recordCoverage(tools: [[parser: 'JACOCO']],
                    // id: 'jacoco', name: 'Coverage',
                    // sourceCodeRetention: 'EVERY_BUILD',
                    // sourceDirectories: [[path: 'src']],
                    // qualityGates: [
                    //     [threshold: 60.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]
                    // ])
                // }
            }
        }    

        stage('Build Docker Image') {
            steps {
                // Adding sleep to ensure Docker Daemon is ready in dind container
                sh 'sleep 10'
                sh 'docker build -t "${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}" ./src'
            }
        }

        stage('Push Docker Image') {
            steps {
                // Adding sleep to ensure Docker Daemon is ready in dind container
                sh 'sleep 10'
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'REGISTRY_USERNAME', passwordVariable: 'REGISTRY_PASSWORD')]) {
                    sh '''
                    echo "${REGISTRY_PASSWORD}" | docker login -u "${REGISTRY_USERNAME}" --password-stdin
                    docker push "${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    '''
                }

                script {
                    image_sha = sh(script: 'docker inspect --format="{{index .RepoDigests 0}}" "${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}" | cut -d "@" -f 2', returnStdout: true).trim()
                }
            }
        }

        stage('Deploy') {
            agent { 
                label 'python' 
            }
            environment {
                IMAGE_SHA = "${image_sha}"
            }
            steps {  
                sh '''
                echo "${IMAGE_SHA}"
                helm upgrade --install --set image.tag="${IMAGE_TAG}" --set image.sha256="${IMAGE_SHA}" -f ./chart/values.yaml dummy-flask-app ./chart
                '''
            }
        }            
    }
}

// recordCoverage(tools: [[parser: 'JACOCO']],
//         id: 'jacoco', name: 'JaCoCo Coverage',
//         sourceCodeRetention: 'EVERY_BUILD',
//         qualityGates: [
//                 [threshold: 60.0, metric: 'LINE', baseline: 'PROJECT', unstable: true],
//                 [threshold: 60.0, metric: 'BRANCH', baseline: 'PROJECT', unstable: true]])


// https://medium.com/geekculture/jenkins-pipeline-python-and-docker-altogether-442d38119484