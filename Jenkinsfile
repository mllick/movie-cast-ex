pipeline {
    agent any

    environment {
        DOCKER_ID = "sarrlick"  
        DOCKER_TAG = "v.${BUILD_ID}.0"
        KUBECONFIG = credentials("config")
        DOCKER_HUB_PASS = credentials("DOCKER_HUB_PASS")  
        PATH="$PATH:~/.local/bin/"

    }

    stages {
        
        stage('Docker Build Cast Service') {
            steps {
                script {
                    sh '''
                        cd cast-service
                        docker build -t ${DOCKER_ID}/cast-service:${DOCKER_TAG} .
                        cd ..
                    '''
                }
            }
        }

        stage('Docker Build Movie Service') {
            steps {
                script {
                    sh '''
                        cd movie-service
                        docker build -t ${DOCKER_ID}/movie-service:${DOCKER_TAG} .
                        cd ..
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_HUB_PASS
                        docker push ${DOCKER_ID}/cast-service:${DOCKER_TAG}
                        docker push ${DOCKER_ID}/movie-service:${DOCKER_TAG}
                    '''
                }
            }
        }

        stage('Deployment et Tests') {
            steps {
                script {
                    sh '''
                        curl localhost:8080

                    '''
                }
            }
        }

        stage('Deploy Cast Service to Dev Environment') {
            steps {
                script {
                sh '''
                    cp mon-chart-helm/values.yaml values.yml
                    helm upgrade --install cast-service-dev ./mon-chart-helm --values=values.yml --namespace dev
                '''
        }
            }
        }
        
        stage('Deploy Movie Service to Dev Environment') {
            steps {
                script {
                    sh '''
                        cp mon-chart-helm/values.yaml values.yml
                        helm upgrade --install movie-service-dev ./mon-chart-helm --values=values.yml --namespace dev
                    '''
                }
            }
        }
    
        stage('Deploy Cast Service to QA Environment') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install cast-service-qa ./mon-chart-helm --values=values.yml --namespace qa
                    '''
                }
            }
        }

        stage('Deploy Movie Service to QA Environment') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install movie-service-qa ./mon-chart-helm --values=values.yml --namespace qa
                    '''
                }
            }
        }
        stage('Deploy Cast Service to Staging Environment') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install cast-service-staging ./mon-chart-helm --values=values.yml --namespace staging
                    '''
                }
            }
        }
    
        stage('Deploy Movie Service to Staging Environment') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install movie-service-staging ./mon-chart-helm --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deploy Cast Service to Prod Environment') {
            steps {
                script {
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install cast-service-prod ./mon-chart-helm --values=values.yml --namespace prod
                    '''
                }
            }
        }
        stage('Deploy Movie Service to Prod Environment') {
            steps {
                script {
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install movie-service-prod ./mon-chart-helm --values=values.yml --namespace prod
                    '''
                }
            }
        }

    
    }
}

