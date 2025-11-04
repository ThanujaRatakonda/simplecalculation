pipeline {
    agent any

    environment {
        IMAGE_NAME = "simplecalculation"
        IMAGE_TAG = "latest"
        REGISTRY = "10.131.103.92:8090"
        PROJECT = "simplecalculation"
        FULL_IMAGE = "${REGISTRY}/${PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ThanujaRatakonda/simplecalculation.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Scan with Trivy') {
            steps {
                sh """
                    trivy image ${IMAGE_NAME}:${IMAGE_TAG} \
                    --format table \
                    --output trivy-report.txt || echo 'Trivy scan failed'
                """
                archiveArtifacts artifacts: 'trivy-report.txt', fingerprint: true
            }
        }

        stage('Login to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "docker login ${REGISTRY} -u $USER -p $PASS"
                }
            }
        }

        stage('Push to Harbor') {
            steps {
                sh """
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE}
                    docker push ${FULL_IMAGE}
                """
            }
        }
    }
}
