pipeline {
    agent any

    environment {
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
                }
            }
        }

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
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'mkdir -p ~/.kube && cat "$KUBECONFIG_FILE" > ~/.kube/config'
                    def namespaces = ['dev', 'qa', 'staging']
                    namespaces.each { namespace ->
                        sh """
                            kubectl create namespace ${namespace} --dry-run=client -o yaml | kubectl apply -f -
                            helm upgrade --install cast-service ./Chart.yaml --namespace ${namespace} --set image.repository=didiiiw/jen,image.tag=cast-service-latest -f .${namespace}-values.yaml
                            helm upgrade --install movie-service ./Chart.yaml --namespace ${namespace} --set image.repository=didiiiw/jen,image.tag=movie-service-latest -f ./${namespace}-values.yaml
                        """
                    }
                }
            }
        }

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
                        helm upgrade --install cast-service ./Chart.yaml --namespace prod --set image.repository=didiiiw/jen,image.tag=cast-service-latest -f ./prod-values.yaml
                        helm upgrade --install movie-service ./Chart.yaml --namespace prod --set image.repository=didiiiw/jen,image.tag=movie-service-latest -f ./prod-values.yaml
                    """
                }
            }
        }
    }
}
