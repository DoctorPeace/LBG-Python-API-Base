pipeline {
    agent any
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
                        echo "main:Build not required in main"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker build -t drpeace/python-api -t drpeace/python-api:dev-v${BUILD_NUMBER} .
                        docker build -t drpeace/flask-nginx -t drpeace/flask-nginx:dev-v${BUILD_NUMBER} ./nginx
                        echo "dev:Build successful"
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
                        echo "main:Push successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker push drpeace/python-api
                        docker push drpeace/python-api:dev-v${BUILD_NUMBER}
                        docker push drpeace/flask-nginx
                        docker push drpeace/flask-nginx:dev-v${BUILD_NUMBER}
                        echo "dev:Push successful"
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
                        kubectl apply -f ./kubernetes --namespace production
                        kubectl set image deployment/flask-deployment flask-container=drpeace/python-api:prod-v${BUILD_NUMBER} -n production
                        echo "main:Build successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        kubectl apply -f ./kubernetes --namespace development
                        kubectl set image deployment/flask-deployment flask-container=drpeace/python-api:dev-v${BUILD_NUMBER} -n development
                        echo "dev:Build successful"
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
                        docker system prune -f
                        echo "main:Cleanup completed"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker system prune -f
                        echo "dev:Cleanup completed"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        echo "Cleanup - Unrecognised branch"
                    }
                } 
            }
        }
    }
}
