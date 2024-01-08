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
        
}
}

