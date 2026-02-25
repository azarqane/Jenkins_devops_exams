pipeline {
    agent any

    environment {
        DOCKER_ID = "azarops"
        MOVIE_IMAGE = "movie-service"
        CAST_IMAGE = "cast-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {
        stage('Docker Build & Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_hub', variable: 'DOCKER_PASS')]) {
                        sh "docker login -u ${DOCKER_ID} -p ${DOCKER_PASS}"

                        sh "docker build -t ${DOCKER_ID}/${MOVIE_IMAGE}:${DOCKER_TAG} ./movie-service"
                        sh "docker push ${DOCKER_ID}/${MOVIE_IMAGE}:${DOCKER_TAG}"

                        sh "docker build -t ${DOCKER_ID}/${CAST_IMAGE}:${DOCKER_TAG} ./cast-service"
                        sh "docker push ${DOCKER_ID}/${CAST_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to DEV') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh "sed -i 's/tag:.*/tag: ${DOCKER_TAG}/g' ./charts/values.yaml" 
                    sh "helm upgrade --install movie-app ./charts --namespace dev --create-namespace"
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install movie-app ./charts --namespace qa --create-namespace"
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install movie-app ./charts --namespace staging --create-namespace"
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                anyOf { branch 'master'; branch 'main' }
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: "Approuver le déploiement en Production ?", ok: "Déployer"
                }
                withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install movie-app ./charts --namespace prod --create-namespace"
                }
            }
        }
    }
}
