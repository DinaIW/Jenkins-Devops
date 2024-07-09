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
                    // Assurez-vous que le fichier kubeconfig est accessible
                    sh '''
                      rm -Rf .kube
                      mkdir .kube
                      ls
                      cat $KUBECONFIG > .kube/config
                      cp fastapi/values.yaml values.yml
                      cat values.yml
                    '''
                    // Exporter le chemin du kubeconfig mis à jour
                    sh 'export KUBECONFIG=/var/lib/jenkins/.kube/config'

                    // Exécutez vos commandes helm ici
                    def environments = [
                        [name: 'dev', valuesFile: 'dev-values.yaml'],
                        [name: 'qa', valuesFile: 'qa-values.yaml'],
                        [name: 'staging', valuesFile: 'staging-values.yaml'],
                        [name: 'prod', valuesFile: 'prod-values.yaml']
                    ]

                    environments.each { env ->
                        sh "helm install ${env.name} . -f ${env.name}-values.yaml --namespace ${env.name}"
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
                    // Assurez-vous que le fichier kubeconfig est accessible
                    sh '''
                        mkdir -p /var/lib/jenkins/.kube
                        cp /var/lib/jenkins/config /var/lib/jenkins/.kube/config
                        chown jenkins:jenkins /var/lib/jenkins/.kube/config
                        chmod 700 /var/lib/jenkins/.kube/config
                    '''
                    // Exporter le chemin du kubeconfig mis à jour
                    sh 'export KUBECONFIG=/var/lib/jenkins/.kube/config'

                    // Exécutez vos commandes kubectl et helm ici
                    sh """
                        kubectl create namespace prod --dry-run=client -o yaml | kubectl apply -f -
                        helm upgrade --install cast-service-prod ./Chart.yaml --namespace prod \
                            --set image.repository=didiiiw/jen,image.tag=cast-service-latest \
                            -f prod-values.yaml
                    """
                }
            }
        }
    }
}
