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
            environment {
                KUBECONFIG = credentials("kubeconfig-credentials")
            }
            steps {
                script {
                    def environments = [
                        [name: 'dev', valuesFile: 'dev-values.yaml'],
                        [name: 'qa', valuesFile: 'qa-values.yaml'],
                        [name: 'staging', valuesFile: 'staging-values.yaml']
                    ]

                    // Prepare Kubeconfig
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                    '''
                    
                    environments.each { env ->
                        sh "helm upgrade ${env.name} . -f ${env.valuesFile} --namespace ${env.name}"
                    }
                }
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            environment {
                KUBECONFIG = credentials("kubeconfig-credentials")
            }
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                    '''
                        sh "helm install prod . -f prod-values --namespace prod"
                    }
                }
            }
        }
    }
}
