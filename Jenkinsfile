pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE = "aazimanishdocker/django-app"
    }
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', credentialsId: 'github-credentials', url: 'https://github.com/AazimAnish/django-devops-pipeline.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push()
                    docker.image("${DOCKER_IMAGE}:${env.BUILD_NUMBER}").push('latest')
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: 'deployment-ec2',
                            transfers: [
                                sshTransfer(
                                    execCommand: """
                                        docker pull ${DOCKER_IMAGE}:latest
                                        docker stop django-app || true
                                        docker rm django-app || true
                                        docker run -d --name django-app -p 8000:8000 ${DOCKER_IMAGE}:latest
                                    """
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}