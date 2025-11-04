pipeline {
    agent any

    environment {
        IMAGE_NAME = "smartcalc"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        HARBOR_URL = "10.131.103.92:8090"
        HARBOR_PROJECT = "simplecalculation"
        FULL_IMAGE = "${HARBOR_URL}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}"
        TRIVY_TEMPLATE_PATH = "/var/lib/jenkins/trivy-templates/junit.tpl" // You can remove this if not using a custom template
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

        stage('Trivy Scan') {
            steps {
                // Run Trivy scan and generate a JUnit XML report
                sh """
                    trivy image ${IMAGE_NAME}:${IMAGE_TAG} \
                    --severity CRITICAL,HIGH \
                    --format junit \
                    --output trivy-report.xml
                """
            }
        }

        stage('Publish Trivy Report') {
            steps {
                // Publish the Trivy JUnit report so Jenkins can display it in the UI
                junit 'trivy-report.xml'
            }
        }

        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-creds', usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                    sh "echo \$HARBOR_PASS | docker login ${HARBOR_URL} -u \$HARBOR_USER --password-stdin"
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE}"
                    sh "docker push ${FULL_IMAGE}"
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE} || true"
            }
        }
    }
}

