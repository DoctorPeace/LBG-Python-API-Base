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
                script {
			        if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        echo "Build not required in main"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker build -t drpeace/python-api -t drpeace/python-api:v${BUILD_NUMBER} .
                        docker build -t drpeace/flask-nginx -t drpeace/flask-nginx:v${BUILD_NUMBER} ./nginx
                        echo "Build successful"
                        '''
                    } else {
                        sh '''
                        echo "Build - Unrecognised branch"
                        '''
                    }
                }
            }
        }
        stage('Push') {
            steps {
                script {
			        if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        echo "Push not required in main"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker push drpeace/python-api
                        docker push drpeace/python-api:v${BUILD_NUMBER}
                        docker push drpeace/flask-nginx
                        docker push drpeace/flask-nginx:v${BUILD_NUMBER}
                        echo "Push successful"
                        '''
                    } else {
                        sh '''
                        echo "Push - Unrecognised branch"
                        '''
                    }
                }    
           }
        }
        stage('Deploy') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        ssh -i ~/.ssh/id_rsa jenkins@10.154.0.53 << EOF
                        docker run -d --name flask-app --network project drpeace/python-api
                        docker run -d -p 80:80 --name nginx --network project drpeace/flask-nginx
                        echo "Main:Build successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        ssh -i ~/.ssh/id_rsa jenkins@10.200.0.16 << EOF
                        docker run -d --name flask-app --network project drpeace/python-api
                        docker run -d -p 80:80 --name nginx --network project drpeace/flask-nginx
                        echo "Dev:Build successful"
                        '''
                    } else {
                        echo "Deploy - Unrecognised branch"
                    }
                }
            }
        }
        stage('Cleanup') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        echo "rmi not required in main"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker system prune -f
                        docker rmi drpeace/python-api:v${BUILD_NUMBER}
                        docker rmi drpeace/flask-nginx:v${BUILD_NUMBER}
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        echo "Cleanup - Unrecognised branch"
                    }
                } 
           }
        }
    }
}
