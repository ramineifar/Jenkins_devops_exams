pipeline {
    environment { // Declaration of environment variables
    DOCKER_ID = "ramineifar" // replace this with your docker-id
    DOCKER_MOVIES_IMAGE = "movie-service"
    DOCKER_CASTS_IMAGE = "cast-movie"
    DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
    }
    agent any // Jenkins will be able to select all available agents
    stages {
        stage(' Docker Build'){ // docker build image stage
            parallel {
                stage('Movies')
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
                stage('Casts')
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
        stage('Docker run'){ // run container from our builded image
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

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
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
                                curl localhost:8001
                                curl localhost:8002
                            '''
                        }
                    }
                }

            }
        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
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
            environment
            {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
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
                helm upgrade --install fastapiapp ./charts --values=values.yml --namespace dev
                '''
                }
            }
        }
        stage('Deploiement en staging'){
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
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
                helm upgrade --install fastapiapp ./charts --values=values.yml --namespace staging
                '''
                }
            }
        }
        stage('Deploiement en prod'){
            when {
                branch 'master'
            }
            environment
            {
                KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
            // Create an Approval Button with a timeout of 15minutes.
            // this require a manuel validation in order to deploy on production environment
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
                helm upgrade --install fastapiapp ./charts --values=values.yml --namespace prod
                '''
                }
            }
        }
    }
    post { // send email when the job has failed
        // ..
        failure {
            echo "This will run if the job failed"
            mail to: "rami_neifar@hotmail.fr",
                 subject: "${env.JOB_NAME} - Build # ${env.BUILD_ID} has failed",
                 body: "For more info on the pipeline failure, check out the console output at ${env.BUILD_URL}"
        }
        // ..
    }
}