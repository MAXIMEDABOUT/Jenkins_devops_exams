pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_USER = 'maxdab'
        GITHUB_REPO = 'https://github.com/MAXIMEDABOUT/Jenkins_devops_exams.git'
    }

    stages {
        }

        stage('Build & Push Docker Images') {
            steps {
                script {
                    def services = ['movie-service', 'cast-service']
                    def tag = env.GIT_BRANCH?.tokenize('/')[-1]?.toLowerCase() ?: "dev"

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
                script {
                    def ns = env.BRANCH_NAME.toLowerCase()
                    def services = ['movie-service', 'cast-service']

                    for (s in services) {
                        sh "helm upgrade --install ${s} ./charts/${s} --namespace ${ns} --create-namespace --set image.repository=$DOCKER_USER/${s} --set image.tag=${ns}"
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
                script {
                    def services = ['movie-service', 'cast-service']

                    for (s in services) {
                        sh "helm upgrade --install ${s} ./charts/${s} --namespace prod --create-namespace --set image.repository=$DOCKER_USER/${s} --set image.tag=prod"
                    }
                }
            }
        }
    }
}
