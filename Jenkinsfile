pipeline {
    agent any
    environment {
        DOCKER_HOST = 'tcp://docker:2375'  // HTTP connection to Docker daemon
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jackbak11/Nginx01-Testing.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t my-nginx-app:${env.BUILD_NUMBER} ."
            }
        }
        
        stage('Test') {
            steps {
                sh 'docker rm -f test-nginx || true'  // Ensure clean state
                sh 'docker run -d --name test-nginx -p 8081:80 my-nginx-app:${env.BUILD_NUMBER}'
                sh 'sleep 5'
                sh 'curl --fail http://localhost:8081 || exit 1'  // Test on host port
                sh 'docker stop test-nginx && docker rm test-nginx'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'docker rm -f my-nginx-app || true'  // Ensure clean state
                sh 'docker run -d --name my-nginx-app -p 80:80 my-nginx-app:${env.BUILD_NUMBER}'
            }
        }
        
        stage('Cleanup') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }
    
    post {
        always {
            cleanWs()  // No node block needed with agent any at pipeline level
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
