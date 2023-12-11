pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                sh '''
                ssh -i ~/.ssh/id_rsa jenkins@10.154.0.53 << EOF
                docker stop flask-app || echo "flask-app not running"
                docker rm flask-app || echo "flask-app not running"
                docker stop nginx || echo "nginx not running"
                docker rm nginx || echo "nginx not running"
                docker rmi drpeace/python-api || echo "Python image does not exist"
                docker rmi drpeace/flask-nginx || echo "Flask image does not exist"
                docker network create project || echo "network already exists"
                '''
           }
        }
        stage('Build') {
            steps {
                sh '''
                docker build -t drpeace/python-api -t drpeace/python-api:v${BUILD_NUMBER} .
                docker build -t drpeace/flask-nginx -t drpeace/flask-nginx:v${BUILD_NUMBER} ./nginx
                '''
           }
        }
        stage('Push') {
            steps {
                sh '''
                docker push drpeace/python-api
                docker push drpeace/python-api:v${BUILD_NUMBER}
                docker push drpeace/flask-nginx
                docker push drpeace/flask-nginx:v${BUILD_NUMBER}
                '''
           }
        }
        stage('Deploy') {
            steps {
                sh '''
                ssh -i ~/.ssh/id_rsa jenkins@10.154.0.53 << EOF
                docker run -d --name flask-app --network project drpeace/python-api
                docker run -d -p 80:80 --name nginx --network project drpeace/flask-nginx
                '''
            }
        }
        stage('Cleanup') {
            steps {
                sh '''
                docker system prune -f
                docker rmi drpeace/python-api:v${BUILD_NUMBER}
                docker rmi drpeace/flask-nginx:v${BUILD_NUMBER}
                '''
           }
        }
    }
}
