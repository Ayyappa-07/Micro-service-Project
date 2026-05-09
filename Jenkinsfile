pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        APP_NAME = "user-service"
        DOCKER_IMAGE = "ayyuraj/user-service:1.0"
        KUBECONFIG = "/var/jenkins_home/.kube/config"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Ayyappa-07/Micro-service-Project.git'
            }
        }

        stage('Build Maven') {
            steps {
                dir('user-service') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('user-service') {
                    sh 'mvn test'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('user-service') {
                    sh '''
                        docker build -t $DOCKER_IMAGE .
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    export KUBECONFIG=$KUBECONFIG
                    kubectl set image deployment/user-service user-service=$DOCKER_IMAGE
                    kubectl rollout status deployment/user-service
                '''
            }
        }
    }

    post {
        success {
            echo "PIPELINE SUCCESS 🚀"
        }

        failure {
            echo "PIPELINE FAILED ❌"
        }
    }
}
