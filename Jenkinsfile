pipeline {
    agent any
    
    environment {
        DOCKERHUB_USERNAME = 'sakit333'
        DOCKER_IMAGE = "${DOCKERHUB_USERNAME}/canara_sak"
        DOCKER_CONTAINER = 'canara_app_sak'
    }

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'prod'], description: 'Select deployment environment')
        choice(name: 'ACTION', choices: ['deploy', 'remove'], description: 'Select action to perform')
    }

    stages {
        stage('Check the docker installed or not'){
            steps {
                sh 'docker --version'
            }
        }
        stage('Run in the Dev Environment') {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'dev' }
                    expression { params.ACTION == 'deploy' }
                }
            }
            steps {
                echo 'Deploying to Development Environment'
                sh '''
                    sudo docker-compose down
                    sudo docker-compose up -d --build
                '''
            }
        }
        stage("Remove container in Dev Environment") {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'dev' }
                    expression { params.ACTION == 'remove' }
                }
            }
            steps {
                echo 'Stopping Development Environment'
                sh 'sudo docker-compose down'
            }
        }
        stage('Build Docker Image') {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'prod' }
                    expression { params.ACTION == 'deploy' }
                }
            }
            steps {
                sh 'sudo docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Login to Docker Hub') {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'prod' }
                    expression { params.ACTION == 'deploy' }
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-cred', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh 'echo $DOCKERHUB_PASS | sudo docker login -u $DOCKERHUB_USER --password-stdin'
                }
            }
        }
        stage('Docker tag with Build ID') {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'prod' }
                    expression { params.ACTION == 'deploy' }
                }
            }
            steps {
                sh "sudo docker tag $DOCKER_IMAGE $DOCKER_IMAGE:${env.BUILD_ID}"
            }
        }
        stage('Push to Dockerhub both latest and build id') {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'prod' }
                    expression { params.ACTION == 'deploy' }
                }
            }
            steps {
                sh 'sudo docker push $DOCKER_IMAGE:${BUILD_ID}'
                sh 'sudo docker push $DOCKER_IMAGE:latest'
            }
        }
        stage('Deploy Docker Container') {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'prod' }
                    expression { params.ACTION == 'deploy' }
                }
            }
            steps {
                sh """
                    sudo docker pull $DOCKER_IMAGE:latest
                    if [ "$(sudo docker ps -q -f name=canara_app_sak)" ]; then
                        sudo docker rm -f canara_app_sak
                    fi
                    sudo docker run -d --name ${DOCKER_CONTAINER} -p 8085:8080 $DOCKER_IMAGE:latest
                """
            }
        }
        stage('Remove Docker Container') {
            when {
                allOf {
                    expression { params.ENVIRONMENT == 'prod' }
                    expression { params.ACTION == 'remove' }
                }
            }
            steps {
                sh '''
                    if [ "$(sudo docker ps -q -f name=canara_app_sak)" ]; then
                        sudo docker rm -f canara_app_sak
                    else
                        echo "Container canara_app_sak is not running."
                    fi
                '''
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline execution failed.'
        }
    }
}