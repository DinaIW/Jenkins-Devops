pipeline {
    agent any

    environment {
<<<<<<< HEAD
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
=======
        DOCKER_HUB_PASS = credentials('dhub')
        KUBECONFIG_FILE = credentials('kubeconfig-credentials')
        GITHUB_CREDENTIALS = credentials('github-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
                // Utiliser les identifiants GitHub pour récupérer le code source
                git credentialsId: 'github-credentials', url: 'https://github.com/DinaIW/examjen.git'
            }
        }

        stage('Build Docker Images') {
            parallel {
                stage('Build cast-service Image') {
                    steps {
                        script {
                            docker.build("didiiiw/jen:cast-service-latest", "-f cast-service/Dockerfile ./cast-service")
                        }
                    }
                }
                stage('Build movie-service Image') {
                    steps {
                        script {
                            docker.build("didiiiw/jen:movie-service-latest", "-f movie-service/Dockerfile ./movie-service")
                        }
                    }
>>>>>>> 7908125307b447570773c44f8aa0e62823e25a59
                }
            }
        }

<<<<<<< HEAD
        stage('Deploy to QA') {
            steps {
                script {
                    deployToNamespace('qa', 'qa-values.yaml')
=======
        stage('Push Docker Images') {
            parallel {
                stage('Push cast-service Image') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'dhub') {
                                docker.image("didiiiw/jen:cast-service-latest").push()
                            }
                        }
                    }
                }
                stage('Push movie-service Image') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'dhub') {
                                docker.image("didiiiw/jen:movie-service-latest").push()
                            }
                        }
                    }
>>>>>>> 7908125307b447570773c44f8aa0e62823e25a59
                }
            }
        }

<<<<<<< HEAD
        stage('Deploy to Staging') {
            steps {
                script {
                    deployToNamespace('staging', 'staging-values.yaml')
=======
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'mkdir -p ~/.kube && cat "$KUBECONFIG_FILE" > ~/.kube/config'
                    def namespaces = ['dev', 'qa', 'staging']
                    namespaces.each { namespace ->
                        sh """
                            kubectl create namespace ${namespace} --dry-run=client -o yaml | kubectl apply -f -
                            helm upgrade --install cast-service ./Chart.yaml --namespace ${namespace} --set image.repository=didiiiw/jen,image.tag=cast-service-latest -f ${namespace}-values.yaml
                            helm upgrade --install movie-service ./Chart.yaml --namespace ${namespace} --set image.repository=didiiiw/jen,image.tag=movie-service-latest -f ${namespace}-values.yaml
                        """
                    }
>>>>>>> 7908125307b447570773c44f8aa0e62823e25a59
                }
            }
        }

<<<<<<< HEAD
        stage('Deploy to Prod') {
            steps {
                script {
                    deployToNamespace('prod', 'prod-values.yaml')
=======
        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
                script {
                    sh """
                        mkdir -p ~/.kube && cat "$KUBECONFIG_FILE" > ~/.kube/config
                        kubectl create namespace prod --dry-run=client -o yaml | kubectl apply -f -
                        helm upgrade --install cast-service ./Chart.yaml --namespace prod --set image.repository=didiiiw/jen,image.tag=cast-service-latest -f prod-values.yaml
                        helm upgrade --install movie-service ./Chart.yaml --namespace prod --set image.repository=didiiiw/jen,image.tag=movie-service-latest -f prod-values.yaml
                    """
>>>>>>> 7908125307b447570773c44f8aa0e62823e25a59
                }
            }
        }
    }
<<<<<<< HEAD

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
=======
>>>>>>> 7908125307b447570773c44f8aa0e62823e25a59
}
