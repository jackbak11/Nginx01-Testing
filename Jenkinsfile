pipeline {
    agent {
        docker {
            image 'docker:20.10' // Use a Docker image with Docker CLI
            args '-v /var/run/docker.sock:/var/run/docker.sock' // Mount Docker socket
        }
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/jackbak11/Nginx01-Testing.git'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("my-nginx-app:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Test') {
            steps {
                sh 'docker run -d --name test-nginx --network jenkins -p 8081:80 my-nginx-app:${env.BUILD_NUMBER}'
                sh 'sleep 5'
                sh 'curl --fail http://test-nginx:80 || exit 1'
                sh 'docker stop test-nginx && docker rm test-nginx'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'docker stop my-nginx-app || true && docker rm my-nginx-app || true'
                sh 'docker run -d --name my-nginx-app --network jenkins -p 80:80 my-nginx-app:${env.BUILD_NUMBER}'
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
