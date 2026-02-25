pipeline {
    agent any

    environment {
        DOCKER_ID = "azarops"
        // On définit les images pour tes deux services
        MOVIE_IMAGE = "movie-service"
        CAST_IMAGE = "cast-service"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {
        stage('Docker Build & Push') {
            steps {
                script {
                    // Connexion à Docker Hub (une seule fois pour les deux services)
                    withCredentials([string(credentialsId: 'docker_hub', variable: 'DOCKER_PASS')]) {
                        sh "docker login -u ${DOCKER_ID} -p ${DOCKER_PASS}"

                        // --- Build & Push MOVIE SERVICE ---
                        // On pointe vers le dossier movie-service
                        sh "docker build -t ${DOCKER_ID}/${MOVIE_IMAGE}:${DOCKER_TAG} ./movie-service"
                        sh "docker push ${DOCKER_ID}/${MOVIE_IMAGE}:${DOCKER_TAG}"

                        // --- Build & Push CAST SERVICE ---
                        // On pointe vers le dossier cast-service
                        sh "docker build -t ${DOCKER_ID}/${CAST_IMAGE}:${DOCKER_TAG} ./cast-service"
                        sh "docker push ${DOCKER_ID}/${CAST_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to DEV') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    // ATTENTION : Vérifie que ton dossier Helm s'appelle bien 'fastapi'
                    // On met à jour le tag dans le fichier values.yaml avant le déploiement
                    sh "sed -i 's/tag:.*/tag: ${DOCKER_TAG}/g' ./movie-service/values.yaml" 
                    
                    // Déploiement avec Helm (ajuste le chemin du chart ./movie-service ou ./fastapi)
                    sh "helm upgrade --install movie-app ./movie-service --namespace dev --create-namespace"
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install movie-app ./movie-service --namespace qa --create-namespace"
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install movie-app ./movie-service --namespace staging --create-namespace"
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                // S'adapte si ta branche s'appelle main au lieu de master
                anyOf { branch 'master'; branch 'main' }
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: "Approuver le déploiement en Production ?", ok: "Déployer"
                }
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install movie-app ./movie-service --namespace prod --create-namespace"
                }
            }
        }
    }
}
