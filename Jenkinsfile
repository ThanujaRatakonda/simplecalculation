pipeline {
    // Define the agent that will run the pipeline. 
    agent any

    // Environment variables for the pipeline, making it easier to manage the image details and credentials.
    environment {
        IMAGE_NAME = "smartcalc"                 // Name of the Docker image.
        IMAGE_TAG = "${env.BUILD_NUMBER}"        // Use Jenkins build number as the Docker image tag.
        HARBOR_URL = "10.131.103.92:8090"        // URL for the Harbor registry.
        HARBOR_PROJECT = "simplecalculation"     // Project name in Harbor registry.
        FULL_IMAGE = "${HARBOR_URL}/${HARBOR_PROJECT}/${IMAGE_NAME}:${IMAGE_TAG}" // Full image path with tag.
        TRIVY_OUTPUT_JSON = "trivy-output.json"  // File path for storing the JSON output from Trivy
    }

    stages {
        // Stage 1: Checkout - Pull the latest code from the GitHub repository.
        stage('Checkout') {
            steps {
                git 'https://github.com/ThanujaRatakonda/simplecalculation.git'
            }
        }

        // Stage 2: Build Docker Image - Build the Docker image from the Dockerfile.
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        // Stage 3: Trivy Scan - Perform a vulnerability scan on the built Docker image using Trivy.
        stage('Trivy Scan') {
            steps {
                // Run Trivy with CRITICAL and HIGH severities and output the results in JSON format.
                sh """
                    trivy image ${IMAGE_NAME}:${IMAGE_TAG} \
                    --severity CRITICAL,HIGH \
                    --format json \
                    -o ${TRIVY_OUTPUT_JSON}
                """
                // Archive the generated JSON report for later inspection.
                archiveArtifacts artifacts: "${TRIVY_OUTPUT_JSON}", fingerprint: true
            }
        }
             stage('Check for Vulnerabilities') {
    steps {
        script {
            // Corrected jq query with a check for vulnerabilities
            def vulnerabilities = sh(script: """
                jq 'if .Vulnerabilities then .Vulnerabilities | map(select(.Severity == "CRITICAL" or .Severity == "HIGH")) | length else 0 end' ${TRIVY_OUTPUT_JSON}
            """, returnStdout: true).trim()

            // If vulnerabilities count is greater than 0, fail the pipeline
            if (vulnerabilities.toInteger() > 0) {
                error "Pipeline failed due to detected CRITICAL/HIGH vulnerabilities!"
            }
        }
    }
}


        // Stage 4: Push to Harbor - Push the successfully built image to the Harbor registry.
        stage('Push to Harbor') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'harbor-creds', usernameVariable: 'HARBOR_USER', passwordVariable: 'HARBOR_PASS')]) {
                    // Login to Harbor registry using the stored credentials.
                    sh "echo \$HARBOR_PASS | docker login ${HARBOR_URL} -u \$HARBOR_USER --password-stdin"
                    // Tag the Docker image for Harbor registry.
                    sh "docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE}"
                    // Push the image to Harbor registry.
                    sh "docker push ${FULL_IMAGE}"
                }
            }
        }

        // Stage 5: Cleanup - Remove the Docker images to free up space on the agent.
        stage('Cleanup') {
            steps {
                // Remove the locally built Docker images to avoid disk space issues.
                sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE} || true"
            }
        }
    }
}

