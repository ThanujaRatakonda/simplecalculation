pipeline {
    agent any

    environment {
        IMAGE_NAME = "smartcalc"
        HARBOR_URL = "10.131.103.92:8090"
        HARBOR_PROJECT = "simplecalculation"
        FULL_IMAGE = "${HARBOR_URL}/${HARBOR_PROJECT}/${IMAGE_NAME}:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/ThanujaRatakonda/simplecalculation.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                    trivy image ${IMAGE_NAME} \
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

        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-creds', usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                    sh 'docker login ${HARBOR_URL} -u $HARBOR_USER -p $HARBOR_PASS'
                    sh 'docker tag ${IMAGE_NAME} ${FULL_IMAGE}'
                    sh 'docker push ${FULL_IMAGE}'
                }
            }
        }
    }
}
