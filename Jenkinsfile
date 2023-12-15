pipeline {
    agent any
    environment {
        GCR_CREDENTIALS_ID = 'jenkins-gcr' // ID in Jenkins credentials
        IMAGE_NAME = 'christaylor-python-api'
        GCR_URL = 'gcr.io/lbg-mea-16'
    }
    stages {
        stage('Init') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        kubectl create namespace production || echo "namespace production already exists"
                        echo "main:Init successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        kubectl create namespace development || echo "namespace development already exists"
                        echo "dev:Init successful"
                        '''
                    } else {
                        sh '''
                        echo "Init - Unrecognised branch"
                        '''
                    }    
                }   
            }
        }
        stage('Build') {
            steps {
                script {
			        // Branch related actions
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        docker build -t drpeace/python-api -t drpeace/python-api:prod-v${BUILD_NUMBER} .
                        docker build -t drpeace/flask-nginx -t drpeace/flask-nginx:prod-v${BUILD_NUMBER} ./nginx                        
                        echo "Build not required in main"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker build -t drpeace/python-api -t drpeace/python-api:dev-v${BUILD_NUMBER} .
                        docker build -t drpeace/flask-nginx -t drpeace/flask-nginx:dev-v${BUILD_NUMBER} ./nginx
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
 			        // Branch related actions
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        docker push drpeace/python-api
                        docker push drpeace/python-api:prod-v${BUILD_NUMBER}
                        docker push drpeace/flask-nginx
                        docker push drpeace/flask-nginx:prod-v${BUILD_NUMBER}
                        echo "Push not required in main"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker push drpeace/python-api
                        docker push drpeace/python-api:dev-v${BUILD_NUMBER}
                        docker push drpeace/flask-nginx
                        docker push drpeace/flask-nginx:dev-v${BUILD_NUMBER}
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
