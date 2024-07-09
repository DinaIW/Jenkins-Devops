pipeline {
    agent any

    environment {
        // Variables d'environnement pour Docker Hub
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials'
        DOCKERHUB_USERNAME = 'didiiiw'
        CAST_SERVICE_IMAGE = 'didiiiw/jen:cast-service-latest'
        MOVIE_SERVICE_IMAGE = 'didiiiw/jen:movie-service-latest'
        
        // Variables d'environnement pour Kubernetes
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig-credentials'
        HELM_CHART_DIR = "${WORKSPACE}"
    }

    stages {
        stage('Preparation') {
            steps {
                // Récupérer les credentials Docker Hub
                withCredentials([usernamePassword(credentialsId: env.DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    // Connexion à Docker Hub
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }

                // Récupérer les credentials Kubernetes
                withCredentials([file(credentialsId: env.KUBECONFIG_CREDENTIALS_ID, variable: 'KUBECONFIG')]) {
                    // Afficher les informations de contexte Kubernetes
                    sh 'kubectl config view'
                }
            }
        }

        stage('Deploy to Dev') {
            steps {
                script {
                    deployToNamespace('dev', 'dev-values.yaml')
                }
            }
        }

        stage('Deploy to QA') {
            steps {
                script {
                    deployToNamespace('qa', 'qa-values.yaml')
                }
            }
        }

        stage('Deploy to Staging') {
            steps {
                script {
                    deployToNamespace('staging', 'staging-values.yaml')
                }
            }
        }

        stage('Deploy to Prod') {
            steps {
                script {
                    deployToNamespace('prod', 'prod-values.yaml')
                }
            }
        }
    }

    post {
        always {
            // Déconnexion de Docker Hub
            sh 'docker logout'
        }
        success {
            echo 'Deployment succeeded!'
        }
        failure {
            echo 'Deployment failed!'
        }
    }
}

def deployToNamespace(namespace, valuesFile) {
    sh """
        helm upgrade --install ${namespace}-cast-service ${env.HELM_CHART_DIR} \
            --namespace ${namespace} \
            -f ${env.HELM_CHART_DIR}/${valuesFile} \
            --set castService.image=${env.CAST_SERVICE_IMAGE}

        helm upgrade --install ${namespace}-movie-service ${env.HELM_CHART_DIR} \
            --namespace ${namespace} \
            -f ${env.HELM_CHART_DIR}/${valuesFile} \
            --set movieService.image=${env.MOVIE_SERVICE_IMAGE}
    """
}
