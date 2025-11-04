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
                sh 'trivy image ${IMAGE_NAME} --format json --output trivy-report.json'
                // Optional: convert to JUnit XML if you have a script
                // sh 'python3 trivy_to_junit.py trivy-report.json > trivy-report.xml'
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

        stage('Publish Trivy Results') {
            steps {
                // If you have converted to JUnit XML
                // junit 'trivy-report.xml'
                echo 'Trivy results published (manual review needed if not converted to JUnit)'
            }
        }
    }
}
