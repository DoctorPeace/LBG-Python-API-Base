pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                sh '''
                ssh -i ~/.ssh/id_rsa jenkins@10.154.0.53 << EOF
                docker stop flask-app || echo "flask-app not running"
                docker rm flask-app || echo "flask-app not running"
                '''
           }
        }
        stage('Build') {
            steps {
                sh '''
                docker build -t drpeace/python-api -t drpeace/python-api:v${BUILD_NUMBER} .
                '''
           }
        }
        stage('Push') {
            steps {
                sh '''
                docker push drpeace/python-api
                docker push drpeace/python-api:v${BUILD_NUMBER}
                '''
           }
        }
        stage('Deploy') {
            steps {
                sh '''
                ssh -i ~/.ssh/id_rsa jenkins@10.154.0.53 << EOF
                docker run -d -p 80:8080 --name flask-app drpeace/python-api
                '''
            }
        }
        stage('Cleanup') {
            steps {
                sh '''
                docker system prune -f
                '''
           }
        }
    }
}
