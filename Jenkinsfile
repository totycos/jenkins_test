pipeline {
    environment {
        DOCKER_ID = "dupertramp"
        DOCKER_IMAGE_MOVIE = "jenkins_exam_movie"
        DOCKER_IMAGE_CAST = "jenkins_exam_cast"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any

    stages {
        stage('Build microservics') {
            parallel {
                stage('Build Movie microservice') {
                    steps {
                        script {
                            sh '''
                            cd movie-service
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG .
                            sleep 4
                            '''
                        }
                    }
                }
                stage('Build Cast microservice') {
                    steps {
                        script {
                            sh '''
                            cd cast-service
                            docker build -t $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG .
                            sleep 4
                            '''
                        }
                    }
                }
            }
        }

        stage('Run microservices') {
            environment {
                NETWORK_NAME = "my_network"
            }
            parallel {
                stage('Setup Network') {
                    steps {
                        script {
                            sh '''
                            docker network inspect $NETWORK_NAME >/dev/null 2>&1 || docker network create $NETWORK_NAME
                            '''
                        }
                    }
                }
                stage('Run Movie DB') {
                    steps {
                        script {
                            sh '''
                            docker rm -f movie_db || true
                            docker run -d --network $NETWORK_NAME --name movie_db -e POSTGRES_USER=movie_db_username -e POSTGRES_PASSWORD=movie_db_password -e POSTGRES_DB=movie_db_dev -v postgres_data_movie:/var/lib/postgresql/data/ postgres:12.1-alpine
                            sleep 4
                            '''
                        }
                    }
                }
                stage('Run Cast DB') {
                    steps {
                        script {
                            sh '''
                            docker rm -f cast_db || true
                            docker run -d --network $NETWORK_NAME --name cast_db -e POSTGRES_USER=cast_db_username -e POSTGRES_PASSWORD=cast_db_password -e POSTGRES_DB=cast_db_dev -v postgres_data_cast:/var/lib/postgresql/data/ postgres:12.1-alpine
                            sleep 4
                            '''
                        }
                    }
                }
                stage('Run Movie microservice') {
                    steps {
                        script {
                            sh '''
                            docker rm -f movie_service || true
                            docker run -d --network $NETWORK_NAME --name movie_service -e DATABASE_URI=postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev -e CAST_SERVICE_HOST_URL=http://cast_service:8000/api/v1/casts/ -p 8001:8000 $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
                            sleep 4
                            '''
                        }
                    }
                }
                stage('Run Cast microservice') {
                    steps {
                        script {
                            sh '''
                            docker rm -f cast_service || true
                            docker run -d --network $NETWORK_NAME --name cast_service -e DATABASE_URI=postgresql://cast_db_username:cast_db_password@cast_db/cast_db_dev -e MOVIE_SERVICE_HOST_URL=http://movie_service:8000/api/v1/movies/ -p 8002:8000 $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
                            sleep 4
                            '''
                        }
                    }
                }
            }
        }

        stage('Test Acceptance') {
            steps {
                script {
                    sh '''
                    echo "Testing Movie Service..."
                    curl -s -o /dev/null -w "%{http_code}" http://localhost:8001/api/v1/movies/docs | grep 200

                    echo "Testing Cast Service..."
                    curl -s -o /dev/null -w "%{http_code}" http://localhost:8002/api/v1/casts/docs | grep 200
                    '''
                }
            }
        }

        stage('Push microservices') {
            parallel {
                stage('Push Movie microservice') {
                    environment {
                        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
                    }
                    steps {
                        script {
                            sh '''
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                            '''
                        }
                    }
                }
                stage('Push Cast microservice') {
                    environment {
                        DOCKER_PASS = credentials("DOCKER_HUB_PASS")
                    }
                    steps {
                        script {
                            sh '''
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
                            '''
                        }
                    }
                }
            }
        }

        // stage('Déploiement en dev') {
        //     environment {
        //         KUBECONFIG = credentials("config")
        //     }
        //     steps {
        //         script {
        //             sh '''
        //             rm -Rf .kube
        //             mkdir .kube
        //             ls
        //             cat $KUBECONFIG > .kube/config
        //             cp fastapi/values.yaml values.yml
        //             cat values.yml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        //             helm upgrade --install app fastapi --values=values.yml --namespace dev
        //             '''
        //         }
        //     }
        // }

        // stage('Déploiement en staging') {
        //     environment {
        //         KUBECONFIG = credentials("config")
        //     }
        //     steps {
        //         script {
        //             sh '''
        //             rm -Rf .kube
        //             mkdir .kube
        //             ls
        //             cat $KUBECONFIG > .kube/config
        //             cp fastapi/values.yaml values.yml
        //             cat values.yml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        //             helm upgrade --install app fastapi --values=values.yml --namespace staging
        //             '''
        //         }
        //     }
        // }

        // stage('Déploiement en prod') {
        //     environment {
        //         KUBECONFIG = credentials("config")
        //     }
        //     steps {
        //         timeout(time: 15, unit: "MINUTES") {
        //             input message: 'Do you want to deploy in production ?', ok: 'Yes'
        //         }
        //         script {
        //             sh '''
        //             rm -Rf .kube
        //             mkdir .kube
        //             ls
        //             cat $KUBECONFIG > .kube/config
        //             cp fastapi/values.yaml values.yml
        //             cat values.yml
        //             sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
        //             helm upgrade --install app fastapi --values=values.yml --namespace prod
        //             '''
        //         }
        //     }
        // }
    }
}
