pipeline {

    agent any

    environment {
        DOCKER_IMAGE = '2004mohan/django-app'
        CONTAINER_NAME = 'django-jenkins-container'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mohantree/django-jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t DOCKERIMAGE:{BUILD_NUMBER} .
                '''
            }
        }

        stage('Run Django Tests') {
            steps {
                sh '''
                    docker run --rm DOCKERIMAGE:{BUILD_NUMBER} \
                    python manage.py test
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "DOCKERPASSWORD"|dockerlogin-u"DOCKER_USERNAME" --password-stdin

                        docker tag DOCKERIMAGE:{BUILD_NUMBER} ${DOCKER_IMAGE}:latest

                        docker push DOCKERIMAGE:{BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker pull ${DOCKER_IMAGE}:latest

                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true

                    docker run -d \
                        --name ${CONTAINER_NAME} \
                        -p 8000:8000 \
                        ${DOCKER_IMAGE}:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed!'
        }
    }
}
