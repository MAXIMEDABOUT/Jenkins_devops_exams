
pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_USER = 'maxdab'
        KUBECONFIG = '/var/lib/jenkins/.kube/config'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'pwd && ls -la && ls -R charts || true'
            }
        }

        stage('Build & Push Docker Images') {
            steps {
                script {
                    def services = ['movie-service', 'cast-service']
                    def tag = env.BRANCH_NAME?.toLowerCase() ?: "dev"

                    for (s in services) {
                        sh "docker build -t $DOCKER_USER/${s}:${tag} ${s}"
                        sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
                        sh "docker push $DOCKER_USER/${s}:${tag}"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes (non-prod)') {
            when {
                not {
                    branch 'master'
                }
            }
            steps {
                dir('.') {
                    script {
                        def ns = env.BRANCH_NAME?.toLowerCase() ?: "dev"
                        def nodePorts = [
                            'movie-service': [
                                dev: 30007,
                                qa: 30009,
                                staging: 30011
                            ],
                            'cast-service': [
                                dev: 30008,
                                qa: 30010,
                                staging: 30012
                            ]
                        ]
                        def services = ['movie-service', 'cast-service']

                        for (s in services) {
                            sh """
                                export KUBECONFIG=${KUBECONFIG}
                                helm upgrade --install ${s} ./charts \
                                  --namespace ${ns} --create-namespace \
                                  --set image.repository=$DOCKER_USER/${s} \
                                  --set image.tag=${ns} \
                                  --set service.nodePort=${nodePorts[s][ns]}
                            """
                        }
                    }
                }
            }
        }

        stage('Manual Prod Deploy') {
            when {
                branch 'master'
            }
            steps {
                input message: "Déployer en production ?"
                dir('.') {
                    script {
                        def nodePorts = [
                            'movie-service': [
                                prod: 30013
                            ],
                            'cast-service': [
                                prod: 30014
                            ]
                        ]
                        def services = ['movie-service', 'cast-service']

                        for (s in services) {
                            sh """
                                export KUBECONFIG=${KUBECONFIG}
                                helm upgrade --install ${s} ./charts \
                                  --namespace prod --create-namespace \
                                  --set image.repository=$DOCKER_USER/${s} \
                                  --set image.tag=prod \
                                  --set service.nodePort=${nodePorts[s]['prod']}
                            """
                        }
                    }
                }
            }
        }
    }
}
