pipeline {
    agent {
        docker {
            image 'docker:20.10'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
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
                    sh 'docker tag my-nginx-app:${env.BUILD_NUMBER} my-nginx-app:latest'
                }
            }
        }
        
        stage('Test') {
            steps {
                sh 'docker network create jenkins || true'
                sh 'docker run -d --name test-nginx --network jenkins -p 8081:80 my-nginx-app:${env.BUILD_NUMBER}'
                sh 'sleep 5'
                sh 'curl --fail http://localhost:8081 || exit 1'
                sh 'docker stop test-nginx && docker rm test-nginx'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'docker network create jenkins || true'
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
            node {
                cleanWs()
            }
        }
        success {
            echo 'Deployment successful!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
