pipeline {
    agent { label 'build' }

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        DOCKER_REPO = "377766724298.dkr.ecr.eu-west-1.amazonaws.com/k8s-pod-restart-info-collector"
    }

    stages {
        stage('Populate vars') {
            steps {
                script {
                    env.BRANCH_NAME = env.CHANGE_BRANCH ?: env.BRANCH_NAME
                    env.ESCAPED_BRANCH = env.BRANCH_NAME.toLowerCase().replaceAll("[^a-zA-Z0-9\\--]", "-")
                    env.DOCKER_TAG = "${DOCKER_REPO}:${env.ESCAPED_BRANCH}-${env.GIT_COMMIT}"
                    env.DOCKER_TAG_LATEST = "${DOCKER_REPO}:${env.ESCAPED_BRANCH}-latest"
                }
            }
        }
        stage('Build Docker image') {
            steps {
                sh """
                    docker build -t "${env.DOCKER_TAG}" .
                    docker tag "${env.DOCKER_TAG}" "${env.DOCKER_TAG_LATEST}"
                """
            }
        }
        stage('Push Docker image') {
            steps {
                sh """
                    docker push "${env.DOCKER_TAG}"
                    docker push "${env.DOCKER_TAG_LATEST}"
                    docker rmi "${env.DOCKER_TAG}" "${env.DOCKER_TAG_LATEST}" || true
                """
            }
        }
        stage('Push Helm chart') {
            steps {
                sh '''
                    helm package ./helm
                    HELM_CHART=$(ls -1 *.tgz)
                    helm push "${HELM_CHART}" oci://public.ecr.aws/f5c8x2g6
                '''
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
