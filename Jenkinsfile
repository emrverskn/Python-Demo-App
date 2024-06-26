pipeline {
    agent any
    environment {
        APP_NAME = "py-demo-app"
        RELEASE = "1.0.0"
        DOCKER_USER = "emrverskn"
        IMAGE_TAG = "v${RELEASE}-b${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}:${IMAGE_TAG}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
        CONTAINER_NAME = "${APP_NAME}"
    }
    stages {
        stage("Test Application") {
            steps {
                script {
                    sh "make test"
                }
            }
        }

        stage('Build App Docker Image') {
            steps {
                echo 'Building App Image'                
                sh 'docker build --force-rm -t "$IMAGE_NAME" -f ./Dockerfile .'
                sh 'docker image ls'
            }
        }

        stage('Push Image to Dockerhub Repo') {
            steps {
                echo 'Pushing App Image to DockerHub Repo'
                withCredentials([string(credentialsId: 'DockerHub-Token', variable: 'DOCKERHUB_TOKEN')]) {
                    sh 'docker login -u $DOCKER_USER -p $DOCKERHUB_TOKEN'
                    sh 'docker push "$IMAGE_NAME"'
                }
            }
        }

        stage ('Deploy the App') {
            steps {
                script {
                    sh 'docker rm -f "$CONTAINER_NAME" || echo "there is no existing container with this name"'
                    sh 'docker run --name "$CONTAINER_NAME" -d -p 5000:5000 "$IMAGE_NAME"'
                }
            }
        }

        stage('Destroy the infrastructure') {
            steps {
                timeout(time:5, unit:'DAYS') {
                    input message:'Approve terminate'
                }   
                script {
                   sh 'docker rm -f "$CONTAINER_NAME"' 
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker rmi -f "$IMAGE_NAME"'
        }
    }
}
