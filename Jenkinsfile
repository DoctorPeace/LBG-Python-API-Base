pipeline {
    agent any
    stages {
        stage('Init') {
            steps {
                script {
                    if (env.GIT_BRANCH == 'origin/main') {
                        sh '''
                        kubectl create ns prod || echo "namespace prod already exists"
                        echo "main:Init successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        kubectl create ns dev || echo "namespace dev already exists"
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
                        docker build -t gcr.io/lbg-mea-16/christaylor-flask-api -t gcr.io/lbg-mea-16/christaylor-flask-api:prod-v${BUILD_NUMBER} .
                        echo "main:Build not required in main"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker build -t gcr.io/lbg-mea-16/christaylor-flask-api -t gcr.io/lbg-mea-16/christaylor-flask-api:dev-v${BUILD_NUMBER} .
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
                        docker push gcr.io/lbg-mea-16/christaylor-flask-api
                        docker push gcr.io/lbg-mea-16/christaylor-flask-api:prod-v${BUILD_NUMBER}
                        echo "main:Push successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker push gcr.io/lbg-mea-16/christaylor-flask-api
                        docker push gcr.io/lbg-mea-16/christaylor-flask-api:dev-v${BUILD_NUMBER}
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
                        kubectl set image deployment/flask-api-deployment flask-api-container=gcr.io/lbg-mea-16/christaylor-flask-api:prod-v${BUILD_NUMBER} -n prod
                        echo "main:Deploy successful"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        kubectl apply -f ./kubernetes -n dev
                        kubectl set image deployment/flask-api-deployment flask-api-container=gcr.io/lbg-mea-16/christaylor-flask-api:dev-v${BUILD_NUMBER} -n dev
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
                        docker rmi gcr.io/lbg-mea-16/christaylor-flask-api:prod-v${BUILD_NUMBER}
                        echo "main:Cleanup completed"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        sh '''
                        docker rmi gcr.io/lbg-mea-16/christaylor-flask-api:dev-v${BUILD_NUMBER}
                        echo "dev:Cleanup completed"
                        '''
                    } else if (env.GIT_BRANCH == 'origin/dev') {
                        echo "Cleanup - Unrecognised branch"
                    }
                }
                sh '''
                docker rmi gcr.io/lbg-mea-16/christaylor-flask-api:latest
                docker system prune -f 
                '''
            }
        }
    }
}
