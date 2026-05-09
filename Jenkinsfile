pipeline {
    agent any

    tools {
        maven 'MAVEN_HOME'
        jdk 'JAVA_HOME'
    }

    environment {
        APP_NAME = "user-service"
        DOCKER_IMAGE = "ayyuraj/user-service:1.0"
        KUBE_DEPLOY = "user-service"
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning GitHub repo"
                git branch: 'main',
                    url: 'https://github.com/Ayyappa-07/Micro-service-Project.git'
            }
        }

        stage('Build Maven') {
            steps {
                dir('user-service') {
                    echo "Building Spring Boot app"
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('user-service') {
                    echo "Building Docker image"
                    sh "docker build -t ${DOCKER_IMAGE} ."
                }
            }
        }

        stage('Docker Push') {
            steps {
                echo "Pushing image to Docker Hub"
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to K3s') {
            steps {
                echo "Deploying to Kubernetes"
                sh """
                    kubectl set image deployment/${KUBE_DEPLOY} ${APP_NAME}=${DOCKER_IMAGE}
                    kubectl rollout status deployment/${KUBE_DEPLOY}
                """
            }
        }
    }

    post {
        success {
            echo "PIPELINE SUCCESS "
        }
        failure {
            echo "PIPELINE FAILED "
        }
    }
}
