pipeline {
    agent any

    environment {
        DOCKER_ID = "sarrlick"  
        DOCKER_TAG = "v.${BUILD_ID}.0"
        KUBECONFIG = credentials("config")
        DOCKER_HUB_PASS = credentials("DOCKER_HUB_PASS")  
        PATH="$PATH:~/.local/bin/"
        MOVIE_CONTAINER_NAME = "movie-service-${BUILD_ID}"
        CAST_CONTAINER_NAME = "cast-service-${BUILD_ID}"

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
        stage('Docker run'){ // run container from our builded image
                steps {
                    script {
                    sh '''
                    docker run -d --name ${MOVIE_CONTAINER_NAME} -p 8082:8080 ${DOCKER_ID}/movie-service:${DOCKER_TAG}
                    sleep 10
                    docker run -d --name ${CAST_CONTAINER_NAME} -p 8083:8080 ${DOCKER_ID}/cast-service:${DOCKER_TAG}
                    sleep 10
                    '''
                    }
                }
            }
        stage('Acceptance Tests Cast Service Dev') {
            steps {
                script {
                    sh '''
                        # Utilisez curl pour accéder à la documentation Swagger/OpenAPI
                        curl -s http://localhost:8080/api/v1/cast/docs > /dev/null
                    '''
                }
            }
        }

        stage('Acceptance Tests Movie Service Dev') {
            steps {
                script {
                    sh '''
                        # Utilisez curl pour accéder à la documentation Swagger/OpenAPI
                        curl -s http://localhost:8080/api/v1/movies/docs > /dev/null
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
        stage('Clean Up Cast Service to Dev Environment') {
            steps {
                script {
                    sh '''
                        kubectl delete pvc -n dev --selector=app=cast-service
                    '''
                }
            }
        }

        stage('Deploy Movie Service to Dev Environment') {
            steps {
                script {
                    sh '''
                        kubectl delete pvc postgres-data-cast -n dev
                        kubectl delete pvc postgres-data-movie -n dev
                        kubectl delete service cast-service -n dev
                        kubectl delete service movie-service -n dev
                        kubectl delete deployment cast-service -n dev
                        kubectl delete deployment movie-service -n dev
                        kubectl delete service nginx -n dev || true
                        kubectl delete deployment nginx -n dev || true
                        kubectl delete deployment postgres-cast-db -n dev || true
                        kubectl delete deployment postgres-movie-db -n dev || true
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
        stage('Clean Up Cast Service to QA Environment') {
            steps {
                script {
                    sh '''
                        kubectl delete pvc -n qa --selector=app=cast-service
                    '''
                }
            }
        }
        stage('Deploy Movie Service to QA Environment') {
            steps {
                script {
                    sh '''
                        kubectl delete pvc postgres-data-cast -n qa
                        kubectl delete pvc postgres-data-movie -n qa
                        kubectl delete service cast-service -n qa
                        kubectl delete service movie-service -n qa
                        kubectl delete deployment cast-service -n qa
                        kubectl delete deployment movie-service -n qa
                        kubectl delete service nginx -n qa || true
                        kubectl delete deployment nginx -n qa || true
                        kubectl delete deployment postgres-cast-db -n qa || true
                        kubectl delete deployment postgres-movie-db -n qa || true
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
        stage('Clean Up Cast Service to Staging Environment') {
            steps {
                script {
                    sh '''
                        kubectl delete pvc -n staging --selector=app=cast-service
                    '''
                }
            }
        }
        stage('Deploy Movie Service to Staging Environment') {
            steps {
                script {
                    sh '''
                        kubectl delete pvc postgres-data-cast -n staging
                        kubectl delete pvc postgres-data-movie -n staging
                        kubectl delete service cast-service -n staging
                        kubectl delete service movie-service -n staging
                        kubectl delete deployment cast-service -n staging
                        kubectl delete deployment movie-service -n staging
                        kubectl delete service nginx -n staging || true
                        kubectl delete deployment nginx -n staging || true
                        kubectl delete deployment postgres-cast-db -n staging || true
                        kubectl delete deployment postgres-movie-db -n staging || true
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
        stage('Clean Up Cast Service to Prod Environment') {
            steps {
                script {
                    sh '''
                        kubectl delete pvc -n prod --selector=app=cast-service
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
                        kubectl delete deployment postgres-cast-db -n prod
                        kubectl delete configmap postgres-cast-config -n prod || true
                        kubectl delete secret postgres-cast-secret -n prod || true
                        kubectl delete deployment postgres-movie-db -n prod
                        kubectl delete configmap postgres-movie-config -n prod || true
                        kubectl delete secret postgres-movie-secret -n prod || true
                        kubectl delete pvc postgres-data-cast -n prod
                        kubectl delete pvc postgres-data-movie -n prod
                        kubectl delete service cast-service -n prod
                        kubectl delete service movie-service -n prod
                        kubectl delete deployment cast-service -n prod
                        kubectl delete deployment movie-service -n prod
                        kubectl delete service nginx -n prod || true
                        kubectl delete deployment nginx -n prod || true
                        kubectl delete deployment postgres-cast-db -n prodg || true
                        kubectl delete deployment postgres-movie-db -n prod || true
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

