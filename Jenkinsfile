pipeline {
    agent any
 
    environment {
        DOCKER_HUB_PASS = credentials('dockerhub-credentials')
        KUBECONFIG_FILE = credentials('kubeconfig-credentials')
        GITHUB_CREDENTIALS = credentials('github-credentials')
    }

    stages {
        stage('Checkout') {
            steps {
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
                            docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
                                docker.image("didiiiw/jen:cast-service-latest").push()
                            }
                        }
                    }
                }
                stage('Push movie-service Image') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials') {
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
                    def environments = [
                        [name: 'dev', valuesFile: 'dev-values.yaml'],
                        [name: 'qa', valuesFile: 'qa-values.yaml'],
                        [name: 'staging', valuesFile: 'staging-values.yaml'],
                        [name: 'prod', valuesFile: 'prod-values.yaml']
                    ]

                    environments.each { env ->
                        sh """
                            kubectl create namespace ${env.name} --dry-run=client -o yaml | kubectl apply -f -
                            helm upgrade --install cast-service-${env.name} ./Chart.yaml --namespace ${env.name} \
                                --set image.repository=didiiiw/jen,image.tag=cast-service-latest \
                                -f ${env.valuesFile}
                            helm upgrade --install movie-service-${env.name} ./Chart.yaml --namespace ${env.name} \
                                --set image.repository=didiiiw/jen,image.tag=movie-service-latest \
                                -f ${env.valuesFile}
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
                        helm upgrade --install cast-service-prod ./Chart.yaml --namespace prod \
                            --set image.repository=didiiiw/jen,image.tag=cast-service-latest \
                            -f prod-values.yaml
                        helm upgrade --install movie-service-prod ./Chart.yaml --namespace prod \
                            --set image.repository=didiiiw/jen,image.tag=movie-service-latest \
                            -f prod-values.yaml
                    """
                }
            }
        }
    }

    post {
        always {
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
