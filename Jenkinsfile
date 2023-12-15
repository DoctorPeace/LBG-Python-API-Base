pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        kubectl create namespace prod || echo "namespace prod already exists"
                        echo "main:Init successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        kubectl create namespace dev || echo "namespace dev already exists"
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
                        docker build -t drpeace/flask-api -t drpeace/flask-api:prod-v${BUILD_NUMBER} .
                        echo "main:Build not required in main"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker build -t drpeace/flask-api -t drpeace/flask-api:dev-v${BUILD_NUMBER} .
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
                        docker push drpeace/flask-api
                        docker push drpeace/flask-api:prod-v${BUILD_NUMBER}
                        echo "main:Push successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker push drpeace/flask-api
                        docker push drpeace/flask-api:dev-v${BUILD_NUMBER}
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
                        kubectl apply -f ./kubernetes -n prod 
                        kubectl set image deployment/flask-deployment flask-api-container=drpeace/python-api:prod-v${BUILD_NUMBER} -n prod
                        echo "main:Deploy successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        kubectl apply -f ./kubernetes -n dev
                        kubectl set image deployment/flask-deployment flask-api-container=drpeace/python-api:dev-v${BUILD_NUMBER} -n dev
                        echo "dev:Deploy successful"
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
                        docker rmi drpeace/python-api:prod-v${BUILD_NUMBER}
                        echo "main:Cleanup completed"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker rmi drpeace/python-api:dev-v${BUILD_NUMBER}
                        echo "dev:Cleanup completed"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        echo "Cleanup - Unrecognised branch"
                    }
                }
                sh '''
                docker system prune -f 
                docker rmi drpeace/python-api
                '''
            }
        }
    }
}
