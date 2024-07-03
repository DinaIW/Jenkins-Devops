pipeline {
    agent any

    environment {
        DOCKER_HUB_PASS = credentials('DOCKER_HUB_PASS')
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
                            docker.build("didiiiw/jen:cast-service-latest", "-f cast-service/Dockerfile .")
                        }
                    }
                }
                stage('Build movie-service Image') {
                    steps {
                        script {
                            docker.build("didiiiw/jen:movie-service-latest", "-f movie-service/Dockerfile .")
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
                            docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_HUB_PASS') {
                                docker.image("didiiiw/jen:cast-service-latest").push()
                            }
                        }
                    }
                }
                stage('Push movie-service Image') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_HUB_PASS') {
                                docker.image("didiiiw/jen:movie-service-latest").push()
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    script {
                        def namespaces = ['dev', 'qa', 'staging', 'prod']
                        namespaces.each { namespace ->
                            sh """
                                kubectl --kubeconfig=$KUBECONFIG create namespace ${namespace} --dry-run=client -o yaml | kubectl apply -f -
                                helm upgrade --install cast-service cast-service-chart --namespace ${namespace} --set image.repository=didiiiw/jen,image.tag=cast-service-latest -f ${namespace}-values.yaml
                                helm upgrade --install movie-service movie-service-chart --namespace ${namespace} --set image.repository=didiiiw/jen,image.tag=movie-service-latest -f ${namespace}-values.yaml
                            """
                        }
                    }
                }
            }
        }
    }
}
