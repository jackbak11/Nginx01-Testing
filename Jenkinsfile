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
                sh label: 'Build', script: 'docker build -t my-nginx-app:${BUILD_NUMBER} .'
            }
        }
        
        stage('Test') {
            steps {
                sh label: 'Remove test container', script: 'docker rm -f test-nginx || true'
                sh label: 'Run test container', script: 'docker run -d --network jenkins --network-alias nginx01 --name test-nginx -p 8081:80 my-nginx-app:${BUILD_NUMBER}'
                sh label: 'Wait', script: 'sleep 5'
                sh label: 'Test curl', script: 'curl --fail http://nginx01 || exit 1'
                sh label: 'Cleanup test container', script: 'docker stop test-nginx && docker rm test-nginx'
            }
        }
        
        stage('Deploy') {
            steps {
                sh label: 'Remove deploy container', script: 'docker rm -f my-nginx-app || true'
                sh label: 'Run deploy container', script: 'docker run -d --name my-nginx-app -p 80:80 my-nginx-app:${BUILD_NUMBER}'
            }
        }
        
        stage('Cleanup') {
            steps {
                sh label: 'Prune images', script: 'docker image prune -f'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
