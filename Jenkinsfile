pipeline {
    environment {
        DOCKER_ID = "ramineifar"
        DOCKER_MOVIES_IMAGE = "movie-service"
        DOCKER_CASTS_IMAGE = "cast-movie"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }
    agent any
    stages {
        stage(' Docker Build'){
            parallel {
                stage('Movies'){
                    steps {
                        script {
                            sh '''
                             docker rm -f movies
                             docker build -t $DOCKER_ID/$DOCKER_MOVIES_IMAGE:$DOCKER_TAG ./movie-service/
                             sleep 6
                            '''
                        }
                    }
                }
                stage('Casts'){
                    steps {
                        script {
                            sh '''
                             docker rm -f casts
                             docker build -t $DOCKER_ID/$DOCKER_CASTS_IMAGE:$DOCKER_TAG ./cast-service/
                             sleep 6
                            '''
                        }
                    }
                }
            }
        }
        stage('Docker run'){
            parallel {
                stage('Movies'){
                    steps {
                        script {
                            sh '''
                            docker run -d -p 8001:8000 --name movies $DOCKER_ID/$DOCKER_MOVIES_IMAGE:$DOCKER_TAG
                            sleep 15
                            '''
                        }
                    }
                }
                stage('Casts'){
                    steps {
                        script {
                            sh '''
                            docker run -d -p 8002:8000 --name casts $DOCKER_ID/$DOCKER_CASTS_IMAGE:$DOCKER_TAG
                            sleep 15
                            '''
                        }
                    }
                }
            }
        }

        stage('Test Acceptance'){
            parallel {
                stage('Movies'){
                    steps {
                        script {
                            sh '''
                            curl localhost:8001
                            '''
                        }
                    }
                }
                stage('Casts'){
                    steps {
                        script {
                            sh '''
                            curl localhost:8002
                            '''
                        }
                    }
                }
            }
        }
        stage('Docker Push'){
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
            parallel {
                stage('Movies') {
                    steps {
                        script {
                            sh '''
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$DOCKER_MOVIES_IMAGE:$DOCKER_TAG
                            '''
                        }
                    }
                }
                stage('Casts') {
                    steps {
                        script {
                            sh '''
                            docker login -u $DOCKER_ID -p $DOCKER_PASS
                            docker push $DOCKER_ID/$DOCKER_CASTS_IMAGE:$DOCKER_TAG
                            '''
                        }
                    }
                }
            }
        }

        stage('Deploiement en dev'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install exam ./fastapiapp --values=values.yml --namespace dev
                    '''
                }
            }
        }

        stage('Deploiement en QA'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install exam ./fastapiapp --values=values.yml --namespace qa
                    '''
                }
            }
        }
        stage('Deploiement en staging'){
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install exam ./fastapiapp --values=values.yml --namespace staging
                    '''
                }
            }
        }
        stage('Deploiement en prod'){
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("config")
            }
            steps {
                timeout(time: 15, unit: "MINUTES") {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                }
                script {
                    sh '''
                    rm -Rf .kube
                    mkdir .kube
                    ls
                    cat $KUBECONFIG > .kube/config
                    cp charts/values.yaml values.yml
                    cat values.yml
                    sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                    helm upgrade --install exam ./fastapiapp --values=values.yml --namespace prod
                    '''
                }
            }
        }
    }
}