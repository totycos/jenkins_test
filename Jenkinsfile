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
            parallel {
                stage('Run Movie microservice') {
                    steps {
                        script {
                            sh '''
                            docker rm -f cast_container || true
                            docker run -d -p 8001:8000 --name movie_container $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                            sleep 4
                            '''
                        }
                    }
                }
                stage('Run Cast microservice') {
                    steps {
                        script {
                            sh '''
                            docker rm -f cast_container || true
                            docker run -d -p 8002:8000 --name cast_container $DOCKER_ID/$DOCKER_IMAGE_CAST:$DOCKER_TAG
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
                    curl localhost
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
