pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Validate configuration') {
            steps {
                sh 'docker compose config --quiet'
                sh 'helm lint charts/'
            }
        }

        stage('Build Docker images') {
            steps {
                sh '''
                    docker build -t movie-service:${BUILD_NUMBER} ./movie-service
                    docker build -t cast-service:${BUILD_NUMBER} ./cast-service
                '''
            }
        }

        stage('Publish Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub_credentials',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_TOKEN" | docker login \
                          --username "$DOCKERHUB_USER" --password-stdin

                        docker tag movie-service:${BUILD_NUMBER} \
                          "$DOCKERHUB_USER/movie-service:${BUILD_NUMBER}"
                        docker tag cast-service:${BUILD_NUMBER} \
                          "$DOCKERHUB_USER/cast-service:${BUILD_NUMBER}"

                        docker push "$DOCKERHUB_USER/movie-service:${BUILD_NUMBER}"
                        docker push "$DOCKERHUB_USER/cast-service:${BUILD_NUMBER}"

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy DEV') {
            steps {
                sh '''
                    helm upgrade --install examen charts \
                      --namespace dev --create-namespace \
                      --set movieService.image.repository="nagiman/movie-service" \
                      --set movieService.image.tag="${BUILD_NUMBER}" \
                      --set castService.image.repository="nagiman/cast-service" \
                      --set castService.image.tag="${BUILD_NUMBER}" \
                      --wait --timeout 5m
                '''
            }
        }

        stage('Deploy QA') {
            when {
                expression {
                    env.BRANCH_NAME == 'master' ||
                    env.GIT_BRANCH == 'origin/master' ||
                    env.GIT_BRANCH == 'master'
                }
            }
            steps {
                sh '''
                    helm upgrade --install examen charts \
                      --namespace qa --create-namespace \
                      --set movieService.image.repository="nagiman/movie-service" \
                      --set movieService.image.tag="${BUILD_NUMBER}" \
                      --set castService.image.repository="nagiman/cast-service" \
                      --set castService.image.tag="${BUILD_NUMBER}" \
                      --wait --timeout 5m
                '''
            }
        }

        stage('Deploy STAGING') {
            when {
                expression {
                    env.BRANCH_NAME == 'master' ||
                    env.GIT_BRANCH == 'origin/master' ||
                    env.GIT_BRANCH == 'master'
                }
            }
            steps {
                sh '''
                    helm upgrade --install examen charts \
                      --namespace staging --create-namespace \
                      --set movieService.image.repository="nagiman/movie-service" \
                      --set movieService.image.tag="${BUILD_NUMBER}" \
                      --set castService.image.repository="nagiman/cast-service" \
                      --set castService.image.tag="${BUILD_NUMBER}" \
                      --wait --timeout 5m
                '''
            }
        }

        stage('Approve PRODUCTION deployment') {
            when {
                expression {
                    env.BRANCH_NAME == 'master' ||
                    env.GIT_BRANCH == 'origin/master' ||
                    env.GIT_BRANCH == 'master'
                }
            }
            steps {
                input message: 'Deploy this build to PRODUCTION?',
                      ok: 'Deploy to production',
                      submitter: 'admin'
            }
        }

        stage('Deploy PRODUCTION') {
            when {
                expression {
                    env.BRANCH_NAME == 'master' ||
                    env.GIT_BRANCH == 'origin/master' ||
                    env.GIT_BRANCH == 'master'
                }
            }
            steps {
                sh '''
                    helm upgrade --install examen charts \
                      --namespace prod --create-namespace \
                      --set movieService.image.repository="nagiman/movie-service" \
                      --set movieService.image.tag="${BUILD_NUMBER}" \
                      --set castService.image.repository="nagiman/cast-service" \
                      --set castService.image.tag="${BUILD_NUMBER}" \
                      --wait --timeout 5m
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline terminé.'
        }
    }
}
