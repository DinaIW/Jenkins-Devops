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

        stage('Login to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v2/', 'dhub') {
                        docker.login(username: 'didiiiw', password: DOCKER_HUB_PASS)
                    }
                }
            }
        }

        stage('Push Docker Images') {
            parallel {
                stage('Push cast-service Image') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v2/', 'dhub') {
                                docker.image("didiiiw/jen:cast-service-latest").push()
                            }
                        }
                    }
                }
                stage('Push movie-service Image') {
                    steps {
                        script {
                            docker.withRegistry('https://index.docker.io/v2/', 'dhub') {
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
                    def namespaces = ['dev', 'qa', 'staging']
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

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                input message: 'Deploy to Production?', ok: 'Deploy'
                withCredentials([file(credentialsId: 'kubeconfig-credentials', variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl --kubeconfig=$KUBECONFIG create namespace prod --dry-run=client -o yaml | kubectl apply -f -
                        helm upgrade --install cast-service cast-service-chart --namespace prod --set image.repository=didiiiw/jen,image.tag=cast-service-latest -f prod-values.yaml
                        helm upgrade --install movie-service movie-service-chart --namespace prod --set image.repository=didiiiw/jen,image.tag=movie-service-latest -f prod-values.yaml
                    """
                }
            }
        }
    }
}
