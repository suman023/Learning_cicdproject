pipeline {
    agent any

    environment {
        IMAGE_NAME = "suman2304/myapp"
        SONAR_PROJECT_KEY = "suman023_Learning_cicdproject"
        SONAR_ORG = "suman023"
        SONAR_TOKEN = "3d6c52810766bd7390ed6ae7cd14211394facf91"
    }

    stages {

        stage('Clone') {
            steps {
                git 'https://github.com/suman023/Learning_cicdproject.git'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | sudo docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                sh '''
                    sudo docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    
                    sudo docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    
                '''
            }
        }

        stage('Sonar Scan') {
            steps {
                sh '''
                    docker run --rm \
                        -e SONAR_HOST_URL="https://sonarcloud.io" \
                        -e SONAR_TOKEN="${SONAR_TOKEN}" \
                        -v "$(pwd):/usr/src" \
                        sonarsource/sonar-scanner-cli \
                        -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                        -Dsonar.organization=${SONAR_ORG} \
                        -Dsonar.sources=.
                '''
            }
        }

        stage('Install Trivy') {
            steps {
                sh '''
                    sudo apt-get install -y wget apt-transport-https gnupg lsb-release

                    wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key \
                        | gpg --dearmor \
                        | sudo tee /usr/share/keyrings/trivy.gpg > /dev/null

                    echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" \
                        | sudo tee /etc/apt/sources.list.d/trivy.list

                    sudo apt-get update
                    sudo apt-get install -y trivy
                    trivy --version
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image \
                        --format json \
                        --output trivy-result.json \
                        --severity HIGH,CRITICAL \
                        ${IMAGE_NAME}:latest
                '''
                archiveArtifacts artifacts: 'trivy-result.json', followSymlinks: false
            }
        }

        stage('Notify') {
            steps {
                mail(
                    to: 'sumanshit023@gmail.com',
                    subject: "Build ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        Job Name  : ${env.JOB_NAME}
                        Build No  : ${env.BUILD_NUMBER}
                        Status    : ${currentBuild.result}
                        Build URL : ${env.BUILD_URL}
                    """
                )

                slackSend(
                    channel: '#jenkinsslackwp',
                    color: 'good',
                    message: "✅ Build Successful: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${env.BUILD_URL}"
                )
            }
        }
    }

    post {
        always {
            sh 'sudo docker logout'
        }
        success {
            slackSend(
                channel: '#jenkinsslackwp',
                color: 'good',
                message: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${env.BUILD_URL}"
            )
        }
        failure {
            slackSend(
                channel: '#jenkinsslackwp',
                color: 'danger',
                message: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER} - ${env.BUILD_URL}"
            )
            mail(
                to: 'sumanshit023@gmail.com',
                subject: "❌ Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "Build failed! Check: ${env.BUILD_URL}"
            )
        }
    }
}
