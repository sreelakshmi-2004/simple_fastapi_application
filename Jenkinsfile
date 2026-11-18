
pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'
        DOCKER_IMAGE          = 'sreelakshmins2004/fastapi-app'
        SONAR_SERVER_NAME     = 'sonarqube_env'
        SONAR_SCANNER_TOOL    = 'sonar-scanner'
        GIT_CREDENTIALS_ID    = 'github-cred'
        MANIFEST_BRANCH       = 'main'
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool "${SONAR_SCANNER_TOOL}"

                    withSonarQubeEnv("${SONAR_SERVER_NAME}") {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=fastapi-app-key \
                            -Dsonar.projectName='fastapi-app-project' \
                            -Dsonar.sources=. \
                            -Dsonar.exclusions=**/venv/**,**/.venv/**,**/__pycache__/**,**/.pytest_cache/**,**/manifests/**
                        """
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh """
                            echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin

                            docker build -t ${DOCKER_IMAGE}:${env.BUILD_NUMBER} -t ${DOCKER_IMAGE}:latest .

                            docker push ${DOCKER_IMAGE}:${env.BUILD_NUMBER}
                            docker push ${DOCKER_IMAGE}:latest

                            docker logout
                        """
                    }
                }
            }
        }

        
stage('Update Deployment Manifest') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: "${GIT_CREDENTIALS_ID}",
            usernameVariable: 'GH_USER',
            passwordVariable: 'GH_TOKEN'
        )]) {
            sh """
                git remote set-url origin https://\${GH_USER}:\${GH_TOKEN}@github.com/sreelakshmi-2004/simple_fastapi_application.git

                git fetch origin ${MANIFEST_BRANCH}

                git checkout -B ${MANIFEST_BRANCH} origin/${MANIFEST_BRANCH}

                sed -i '/name: fastapi-app/{n;s|image:.*|image: ${DOCKER_IMAGE}:${env.BUILD_NUMBER}|}' manifests/deployment.yaml

                git config user.name "sreelakshmi-2004"
                git config user.email "sreelakshmins873@gmail.com"

                git add manifests/deployment.yaml

                git commit -m "Update FastAPI image to build ${env.BUILD_NUMBER} [skip ci]" || echo "No changes to commit"

                git push origin ${MANIFEST_BRANCH}
            """
        }
    }
}
    }


    post {
        success {
            echo 'Pipeline executed successfully! FastAPI image built, pushed, and deployment manifest updated.'
        }

        failure {
            echo 'Pipeline failed. Please review the stage logs.'
        }
    }
}
