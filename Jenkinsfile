
pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        MOVIE_IMAGE = 'movie-service'
        CAST_IMAGE  = 'cast-service'
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

        stage('Publier vers DockerHub') {
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
    }

    post {
        always {
            echo 'Pipeline terminé ! '
        }
    }
}
