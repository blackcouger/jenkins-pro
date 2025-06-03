
pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "your-dockerhub/my-app"
        K8S_NAMESPACE = "dev"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                url: 'https://github.com/your-username/my-app.git'
            }
        }
        
        stage('Build & Test') {
            agent {
                docker {
                    image 'python:3.9'
                    args '-v $PWD:/app'
                }
            }
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pytest'  # Your test command
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-creds') {
                        docker.image("${DOCKER_IMAGE}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    kubectl config use-context your-k8s-context
                    kubectl set image deployment/my-app \
                    app=${DOCKER_IMAGE}:${BUILD_NUMBER} -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
        success {
            slackSend(
                channel: '#devops',
                message: "✅ Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
    }
}
