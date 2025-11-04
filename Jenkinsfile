pipeline {
    agent any

    environment {
        IMAGE_NAME = "login-application"
        IMAGE_TAG = "latest"
        REGISTRY = "10.131.103.92:8090"
        PROJECT = "harbor-login"
        FULL_IMAGE = "${REGISTRY}/${PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ThanujaRatakonda/login-application.git'
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

        stage('Publish Trivy Report') {
            steps {
                sh 'echo "<html><body><pre>" > trivy-report.html'
                sh 'cat trivy-report.txt >> trivy-report.html'
                sh 'echo "</pre></body></html>" >> trivy-report.html'
                script {
                    publishHTML(target: [
                        reportDir: '.',
                        reportFiles: 'trivy-report.html',
                        reportName: 'Trivy Vulnerability Report'
                    ])
                }
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
