pipeline {
    agent any

    environment {
        DOCKER_ID = "azarops"
        DOCKER_IMAGE = "examen-jenkins"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {
        stage('Docker Build & Push') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    
                    withCredentials([string(credentialsId: 'docker_hub', variable: 'DOCKER_PASS')]) {
                        sh "docker login -u ${DOCKER_ID} -p ${DOCKER_PASS}"
                        sh "docker push ${DOCKER_ID}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }

        stage('Deploy to DEV') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "sed -i 's/tag:.*/tag: ${DOCKER_TAG}/g' fastapi/values.yaml"
                    sh "helm upgrade --install examen-app ./fastapi --namespace dev --create-namespace"
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install examen-app ./fastapi --namespace qa --create-namespace"
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install examen-app ./fastapi --namespace staging --create-namespace"
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'master'
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: "Approuver le déploiement en Production ?", ok: "Déployer"
                }
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "helm upgrade --install examen-app ./fastapi --namespace prod --create-namespace"
                }
            }
        }
    }
}
